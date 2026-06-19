# Splunk SOC Alert Triage Lab

This repository documents a beginner SOC alert triage workflow using Splunk.

## Objective

Practice how a SOC analyst reviews an alert, validates the evidence, searches related events, and writes clear investigation notes.

## Scenario

A security alert is generated from suspicious or repeated activity. The analyst must decide whether the alert is likely benign, suspicious, or needs escalation.

## Tools Used

- Splunk Enterprise
- Sample security logs
- SPL searches
- Analyst notes

## Triage Workflow

1. Review the alert name and trigger condition.
2. Identify the affected host, user, source IP, and timestamp.
3. Search related events before and after the alert.
4. Check for repeated patterns or unusual behavior.
5. Document findings clearly.
6. Decide whether to close, monitor, or escalate.

## Example SPL Searches

```spl
index=* sourcetype=* error OR failed OR denied
```

```spl
index=* user=* 
| stats count by user, host, src_ip
| sort - count
```

```spl
index=* src_ip=*
| stats count by src_ip
| sort - count
```

## Analyst Notes Template

```text
Alert Name:
Date/Time:
Host:
User:
Source IP:
Observed Activity:
Evidence Found:
Assessment:
Recommended Action:
```

## Skills Practiced

- Alert review
- Splunk searching
- Timeline analysis
- Pattern recognition
- Investigation documentation
- Escalation decision-making

## Next Improvements

- Add screenshots of the alert workflow
- Add sample events
- Add completed analyst notes
- Add severity levels
- Build a small dashboard for alert counts
