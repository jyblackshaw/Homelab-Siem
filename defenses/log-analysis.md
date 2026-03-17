# Log Analysis & Alerting

> **Documentation In Progress**

Documentation of Splunk SPL queries, correlation searches, and alert dashboards used to detect attacks performed in the lab.

## 1. Account Lockout Detection

### Audit Policy Configuration

By default, Windows does not log many of the events needed for security monitoring. Advanced audit policies were enabled on the Domain Controller so that key events like account lockouts (Event ID 4740) are actually generated and forwarded to Splunk.

The following audit categories were enabled (Success and Failure):

| Category | Subcategory | Purpose |
|---|---|---|
| Account Management | All | Tracks user/group creation, deletion, modifications, and lockouts (4740) |
| Account Logon | Credential Validation | Tracks credential validation against the DC (4776) |
| Logon/Logoff | Logon, Logoff, Account Lockout | Tracks failed logon attempts (4625), successful logons (4624), and lockouts |
| Detailed Tracking | Process Creation | Tracks every process spawn (4688) - critical for detecting malware execution |
| Policy Change | Audit Policy Change | Detects if an attacker modifies or disables audit policies to cover their tracks |
| Privilege Use | Sensitive Privilege Use | Tracks use of sensitive privileges like SeDebugPrivilege, commonly abused by tools like Mimikatz |

![Advanced audit policies on DC](../links/screenshots/splunk-queries/advanced-audit-policies.png)

### SPL Query

With the audit policies in place, the following SPL query searches Windows Security event logs for account lockout events:

```spl
index=main sourcetype="WinEventLog:Security" EventCode=4740
```

This filters for Event ID 4740 (account lockout). In a production environment this would be paired with a Splunk alert set to trigger on a short interval (e.g. every 5 minutes) to enable near-real-time detection.

To test the query, the domain account `m.webb` was intentionally locked out by entering incorrect passwords on the W11-Client workstation.

![Account locked out on DC](../links/screenshots/splunk-queries/acc-lockout.png)

### Result

The query returned a lockout event for the user `m.webb`, confirming that the audit policies, log forwarding, and Splunk query are all working end to end.

![Event 4740 - m.webb locked out](../links/screenshots/splunk-queries/win-event-4740.png)

## Sections to Cover
- SPL queries for detecting brute force, command injection, and lateral movement
- Correlation searches linking events across endpoints
- Alert dashboard configuration and thresholds
- Example alerts triggered by attack exercises
- Windows Alert Analysis
- Wireshark Packet Analysis (Windows Event Forwarding to Domain Controller)
