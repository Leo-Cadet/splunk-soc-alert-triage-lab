# Day 8 — Splunk Start, Login Recovery, Index Creation, and Auth Log Prep

## Date

June 2, 2026

## Project Context

This lab continues the Splunk/SOC lab build. Day 6 focused on rebuilding the Kali lab machine and preparing for Splunk. Day 8 moved the project forward by getting Splunk running again, logging into the web interface, checking indexes, creating a new lab index, and preparing fake authentication log data for future detection practice.

The goal was to move from setup into an actual SOC-style workflow: get Splunk operational, prepare an index, and create authentication logs that can be ingested and searched later.

## Lab Environment

- **Operating system:** Kali Linux
- **Tool:** Splunk Enterprise
- **Splunk path:** `/opt/splunk/`
- **Splunk Web:** port `8000`
- **Lab index created:** `linux_lab`
- **Test log path:** `~/splunk-lab/logs/auth-test.log`

## What Happened

### 1. Splunk was not running

I checked Splunk status with:

```bash
sudo /opt/splunk/bin/splunk status
```

Splunk reported that it was not running.

### 2. Started Splunk

I attempted to start Splunk:

```bash
sudo /opt/splunk/bin/splunk start
```

Splunk warned that running as root is deprecated and that the `--run-as-root` option should be used when running as root.

The command that moved the lab forward was:

```bash
sudo /opt/splunk/bin/splunk start --accept-license --run-as-root
```

Splunk performed prerequisite checks, validated configuration, checked indexes, started the daemon, and made the web server available.

### 3. Splunk Web became available

Splunk reported the web interface at:

```text
http://attkcybr2000:8000
```

This confirmed that Splunk was running and reachable through the browser.

## Login Issue

The first login attempt failed.

This became a troubleshooting step. I worked through how to regain access and successfully logged into the Splunk interface.

## Security Question Raised

A major question came up during this lab:

> If someone had my root password, could they do the same thing?

The answer is yes. If an attacker has root access on the Linux machine, they can potentially control Splunk, modify files, reset access, start or stop services, manipulate logs, and interfere with monitoring.

This is an important SOC lesson because admin recovery steps can also show what an attacker could abuse if credentials are compromised.

## Splunk Preferences Updated

After logging in, I updated Splunk preferences. This included search editor settings to make the environment easier to work in while learning.

## Index Review

I viewed the existing indexes in Splunk.

This helped me understand that Splunk uses indexes as storage locations for searchable event data.

Default/internal indexes visible included examples such as:

```text
_audit
_internal
_introspection
_metrics
main
summary
```

## New Index Created

I created a new index for the lab:

```text
linux_lab
```

This index will be used for Linux authentication and SSH-style log testing.

## Fake Authentication Log Created

I created fake authentication data in a local log file:

```bash
nano ~/splunk-lab/logs/auth-test.log
```

The log data included failed and accepted password events so that future searches can practice common SOC-style detection questions.

Example investigation goals for the next lab:

- Find failed passwords
- Identify source IP addresses
- Count failed attempts
- Identify usernames being targeted
- Compare accepted vs failed logins
- Build a basic brute-force style detection search

## Current Status

Completed:

- Checked Splunk status
- Started Splunk successfully
- Accepted license
- Confirmed web server availability on port 8000
- Troubleshot login issue
- Logged into Splunk
- Updated search preferences
- Reviewed indexes
- Created new `linux_lab` index
- Created fake auth log file for future ingestion

Next:

- Add `auth-test.log` as a Splunk data input
- Send the data to the `linux_lab` index
- Run first searches against the lab data
- Search for `Failed password`
- Count failed attempts by user and source IP

## Commands Used

```bash
sudo /opt/splunk/bin/splunk status
sudo /opt/splunk/bin/splunk start
sudo /opt/splunk/bin/splunk start --accept-license --run-as-root
nano ~/splunk-lab/logs/auth-test.log
```

## Key Lesson

Getting Splunk installed is not the finish line. The real SOC work starts when data is indexed and searched.

Today was about getting Splunk ready to work.

Tomorrow is about making Splunk analyze logs.

## Portfolio Value

This lab shows progress from installation into operational use. It documents Splunk service management, web access, login troubleshooting, index awareness, index creation, and preparation of authentication log data for security analysis.

This is a practical bridge from tool setup to SOC investigation work.
