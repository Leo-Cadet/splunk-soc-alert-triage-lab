# Day 10 — Suspicious SSH Successful Login Investigation

## Date

June 4, 2026

## Project Context

This lab continues the Splunk/SOC investigation workflow. Day 9 focused on getting authentication logs into Splunk, troubleshooting search issues, finding failed SSH login activity, extracting source IPs and usernames, and identifying `root` as the most targeted account.

Day 10 moved the investigation forward by asking a stronger SOC question:

> Did any IPs with failed login activity also have successful logins?

This changed the lab from basic failed-login counting into a more realistic investigation sequence: failed attempts, successful authentication, user targeting, and post-login activity.

## Lab Environment

- **Operating system:** Kali Linux
- **Tool:** Splunk Enterprise
- **Data type:** Linux SSH/authentication logs
- **Use case:** Suspicious SSH login investigation
- **Primary focus:** Source IP `192.168.1.90`

## Starting Point

I started by checking whether Splunk was running:

```bash
sudo /opt/splunk/bin/splunk status
```

Splunk was not running:

```text
splunkd is not running.
```

I started Splunk from memory using:

```bash
sudo /opt/splunk/bin/splunk start --run-as-root
```

Splunk started successfully, performed prerequisite checks, validated indexes, and brought the web interface back online.

This was an important progress point because I was able to get Splunk running quickly without needing to look everything up.

## Investigation Question

Yesterday I found that `root` was the most targeted account and had the most failed attempts.

Today I wanted to know:

> Did any source IPs with failed login activity also have successful logins?

That is a stronger analyst question because failed attempts alone do not prove compromise. But failed attempts followed by a successful login can indicate possible credential guessing or unauthorized access.

## Search 1 — Failed and Successful Logins by Source IP

```spl
index=* ("Failed password" OR "Accepted password")
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| rex "Failed password for (invalid user )?(?<failed_user>\w+)"
| rex "Accepted password for (?<success_user>\w+)"
| eval user=coalesce(failed_user, success_user)
| stats 
    count(eval(searchmatch("Failed password"))) as failed_logins
    count(eval(searchmatch("Accepted password"))) as successful_logins
    values(user) as users_seen
    by src_ip
| where failed_logins > 0 OR successful_logins > 0
| sort - failed_logins
```

## Issue: No Results at First

The first search returned no results.

Instead of immediately looking everything up, I thought through the issue and changed the time range. After expanding the time range, results appeared.

## Key Lesson: Time Range Matters

No results does not always mean the SPL is wrong.

It can also mean:

- wrong time range
- wrong index
- wrong source
- wrong sourcetype
- data did not land where expected
- field extraction did not match the raw logs

In this case, the time range was the issue.

## Understanding rex

During this lab, I clarified that `rex` is not the condition.

`rex` means:

> Pull a piece of text out of the raw log and turn it into a field.

Example:

```spl
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
```

Plain meaning:

> Find the word `from`, grab the IP address after it, and save it as a field named `src_ip`.

Simple rule:

```text
search = find matching logs
rex = extract fields from raw text
eval = create or combine fields
stats = count or group results
where = filter final results
sort = order the table
```

## Search 2 — Timeline for Risky IPs

After reviewing the failed and successful login results, I focused on specific risky IPs.

```spl
index=* ("Failed password" OR "Accepted password")
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| rex "Failed password for (invalid user )?(?<failed_user>\w+)"
| rex "Accepted password for (?<success_user>\w+)"
| eval user=coalesce(failed_user, success_user)
| search src_ip IN ("192.168.1.200","10.10.10.5","172.16.5.30","192.168.1.85","192.168.1.90")
| table _time src_ip user _raw
| sort _time
```

This helped move from summary counts into event timeline review.

## Search 3 — Isolating Source IP 192.168.1.90

The most interesting pattern came from `192.168.1.90`.

```spl
index=* ("Failed password" OR "Accepted password")
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| rex "Failed password for (invalid user )?(?<failed_user>\w+)"
| rex "Accepted password for (?<success_user>\w+)"
| eval user=coalesce(failed_user, success_user)
| search src_ip="192.168.1.90"
| table _time src_ip user _raw
| sort _time
```

## Initial SOC Ticket Draft

**Severity:** Medium to High

**Reason:** Failed SSH attempts followed by a successful login from the same source IP.

An abnormal SSH pattern was seen from source IP `192.168.1.90`. The source had failed login attempts against the users `john` and `johnny`, then successfully authenticated as `john`. This may indicate username guessing or credential-based access. Recommended follow-up actions include validating whether the IP belongs to an authorized workstation, confirming the user activity, and reviewing additional post-login commands for the `john` account.

## Follow-Up Question

After identifying a successful login, the next question was:

> What did John do after login?

This is important because authentication is only one part of the investigation. Post-login behavior can show whether the session looked normal or suspicious.

## Search 4 — John Post-Login Activity

```spl
index=* ("john" OR "sudo:")
| rex "sudo:\s(?<sudo_user>\w+)"
| rex "COMMAND=(?<command>.*)"
| table _time sudo_user command _raw
| sort _time
```

This search showed sudo activity connected to the `john` account.

## Updated SOC Finding

Source IP `192.168.1.90` generated failed SSH login attempts against `john` and `johnny`, then successfully authenticated as `john`. Shortly after login, the `john` account executed `sudo whoami`, indicating an attempt to verify elevated/root execution context. This pattern suggests possible credential guessing followed by privilege validation.

## Search 5 — Full John Isolation Search

To keep the investigation focused, I isolated both `john` and `192.168.1.90`.

```spl
index=* ("john" OR "192.168.1.90")
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| rex "Failed password for (invalid user )?(?<failed_user>\w+)"
| rex "Accepted password for (?<success_user>\w+)"
| rex "sudo:\s(?<sudo_user>\w+)"
| rex "COMMAND=(?<command>.*)"
| eval user=coalesce(failed_user, success_user, sudo_user)
| table _time src_ip user command _raw
| sort _time
```

This search combined authentication activity and sudo command activity into one timeline.

## Final SOC Finding

A suspicious SSH authentication sequence was identified from source IP `192.168.1.90`. The host attempted failed logins against `john` and `johnny`, then successfully authenticated as `john`. Shortly after the successful login, the `john` account executed:

```text
sudo /usr/bin/whoami
```

This indicates a possible privilege validation step. The activity is consistent with username guessing followed by successful credential-based access and should be reviewed as a potential unauthorized login.

## Recommended Analyst Actions

- Confirm whether `192.168.1.90` is an authorized workstation
- Confirm whether `john` was expected to log in at that time
- Review additional commands run by `john`
- Check for additional successful logins from the same source IP
- Check whether the same source IP targeted other users
- Review sudo activity around the same time window
- Consider severity Medium to High depending on asset criticality and user confirmation

## What I Learned

1. Starting Splunk from memory shows real command-line progress.
2. No-result searches do not always mean the SPL is wrong.
3. Time range is one of the first things to check in Splunk.
4. `rex` extracts fields; it is not the condition.
5. Failed login counts are useful, but failed logins followed by success are more important.
6. Authentication activity should be followed by post-login command review.
7. `sudo whoami` after login can indicate privilege validation behavior.
8. A good SOC note should explain what happened, why it matters, and what to check next.

## Portfolio Value

This lab shows progression from basic searching into a full mini-investigation. It includes service recovery, SPL troubleshooting, field extraction, event correlation, timeline review, post-login activity review, and a final SOC finding.

This is the clearest SOC-style workflow so far in the lab series.
