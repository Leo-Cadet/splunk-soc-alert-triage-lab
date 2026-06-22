# SSH Authentication Incident Report

## Ticket Title

Suspicious SSH Authentication Pattern - Failed Logins Followed by Successful Access

## Severity

High

## Alert Name

SSH Failed Logins Followed by Successful Login

## Detection Summary

A Splunk scheduled alert was created to detect source IPs with multiple failed SSH login attempts followed by successful authentication. This behavior may indicate credential guessing, password spraying, or unauthorized access using valid credentials.

## Key Evidence

| Evidence Type | Value |
|---|---|
| Source IP | `192.168.1.90` |
| Failed usernames | `john`, `johnny` |
| Successful login | `john` |
| Post-login command | `sudo /usr/bin/whoami` |
| Detection type | Failed SSH logins followed by successful SSH login |

## Timeline

| Time | Source IP | User | Event |
|---|---|---|---|
| 08:32:55 | `192.168.1.90` | `john` | Failed password for invalid user john |
| 08:33:22 | `192.168.1.90` | `johnny` | Failed password for invalid user johnny |
| 08:34:49 | `192.168.1.90` | `john` | Accepted password for john |
| 08:35:18 | N/A | `john` | sudo command executed: `/usr/bin/whoami` |

## Analyst Assessment

The activity shows failed login attempts against related usernames followed by successful authentication as `john`. Shortly after the successful login, the account executed `sudo /usr/bin/whoami`.

The `whoami` command is not malicious by itself, but in this context it may represent privilege validation after suspicious access. This pattern should be treated as suspicious until the login is confirmed as legitimate.

## Risk Explanation

This event sequence may indicate:

- Username guessing
- Credential compromise
- Password spraying
- Unauthorized SSH access
- Post-login privilege checking

## Recommended Actions

1. Confirm whether `192.168.1.90` is an authorized workstation.
2. Validate whether the `john` login was expected.
3. Review all commands executed by `john` after login.
4. Reset the `john` password if the activity was unauthorized.
5. Review SSH configuration and authentication controls.
6. Disable direct root login if enabled.
7. Consider account lockout, MFA, or stronger SSH access controls.

## Status

Investigation Required

## Final Classification

Suspicious Authentication Activity
