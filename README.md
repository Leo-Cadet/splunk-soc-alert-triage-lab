# Splunk SOC Alert Triage Lab

## Project Summary

This project documents a Splunk-based SOC investigation workflow for suspicious Linux SSH authentication activity. The lab uses simulated Linux authentication logs to detect failed SSH logins, targeted usernames, successful logins after failed attempts, and sudo activity after authentication.

The final deliverable is a Splunk dashboard, a scheduled alert, SPL detection searches, and an incident report for a suspicious SSH authentication sequence.

## Objective

Build a practical SOC analyst workflow that answers four investigation questions:

1. Which source IPs generated the most failed SSH login attempts?
2. Which usernames were targeted the most?
3. Did any source IP have failed logins followed by a successful login?
4. Did any risky user run sudo commands after login?

## Lab Scenario

A Linux host generated SSH authentication events. Several source IPs attempted failed logins. One suspicious sequence involved source IP `192.168.1.90`, failed username attempts against `john` and `johnny`, a successful login as `john`, and a sudo command execution of `/usr/bin/whoami`.

This pattern may indicate username guessing followed by successful credential-based access and privilege validation.

## Tools Used

- Splunk Enterprise
- Kali Linux
- Linux authentication-style logs
- SPL searches
- Splunk dashboard panels
- Splunk scheduled alerting
- SOC-style incident reporting

## Skills Demonstrated

- Splunk dashboarding
- SPL search building
- Field extraction with `rex`
- Aggregation with `stats`
- Conditional logic with `eval` and `where`
- SSH authentication analysis
- Failed login investigation
- Failed-to-successful login detection
- Sudo command review
- Alert creation
- SOC ticket writing

## Dashboard Panels Built

| Panel | Purpose |
|---|---|
| Top Failed SSH Login Source IPs | Identifies the source IPs generating the most failed SSH login attempts. |
| Most Targeted SSH Usernames | Identifies accounts attackers or scanners are attempting to access. |
| IPs With Failed and Successful SSH Logins | Finds source IPs that had both failed and successful authentication events. |
| Sudo Command Timeline by Risky Users | Reviews post-login sudo activity for suspicious or risky accounts. |
| Alert Candidates - Failed to Successful SSH Login | Shows source IPs that match the scheduled alert logic. |

## Key Finding

A suspicious SSH authentication sequence was identified:

| Time | Source IP | User | Event |
|---|---|---|---|
| 08:32:55 | 192.168.1.90 | john | Failed login for invalid user `john` |
| 08:33:22 | 192.168.1.90 | johnny | Failed login for invalid user `johnny` |
| 08:34:49 | 192.168.1.90 | john | Successful login as `john` |
| 08:35:18 | N/A | john | Sudo command: `/usr/bin/whoami` |

## Analyst Assessment

The activity from `192.168.1.90` is suspicious because it shows failed login attempts against similar usernames, followed by a successful login and sudo command execution. The `whoami` command is not destructive by itself, but after suspicious authentication activity it may indicate privilege validation.

Severity: **High**

Reason: Failed SSH login attempts followed by successful access and sudo execution.

## Scheduled Alert Created

Alert name:

```text
SSH Failed Logins Followed by Successful Login
```

Detection logic:

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
| where failed_logins >= 2 AND successful_logins >= 1
| sort - failed_logins
```

Trigger condition:

```text
Number of results is greater than 0
```

## Recommended Response Actions

1. Confirm whether `192.168.1.90` is an authorized internal workstation.
2. Validate whether the `john` login was legitimate.
3. Review all commands executed by `john` after login.
4. Reset the `john` password if the activity is unauthorized.
5. Review SSH hardening controls.
6. Disable direct root SSH login where applicable.
7. Consider account lockout or MFA for SSH access.

## Repository Structure

```text
.
├── README.md
├── searches/
│   ├── failed-login-source-ips.spl
│   ├── targeted-usernames.spl
│   ├── failed-to-successful-login-alert.spl
│   ├── john-incident-timeline.spl
│   └── sudo-command-timeline.spl
├── incident-report/
│   └── ssh-authentication-incident-report.md
├── sample-logs/
│   └── auth-test.log
└── screenshots/
    └── README.md
```

## Notes

This lab uses simulated authentication logs for training and portfolio demonstration. The workflow mirrors real SOC triage logic: detect suspicious activity, validate evidence, build a timeline, assess risk, and document recommended response actions.
