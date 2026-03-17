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

To test querying, the domain account `m.webb` was intentionally locked out by entering incorrect passwords on the W11-Client workstation.

![Account locked out on DC](../links/screenshots/splunk-queries/acc-lockout.png)

With the audit policies in place, the following SPL query searches Windows Security event logs for account lockout events:

```spl
index=main sourcetype="WinEventLog:Security" EventCode=4740
```

This filters for Event ID 4740 (account lockout). In a production environment this would be paired with a Splunk alert set to trigger on a short interval (e.g. every 5 minutes) to enable near-real-time detection.

### Result

The query returned a lockout event for the user `m.webb`, confirming that the audit policies, log forwarding, and Splunk query are all working end to end.

![Event 4740 - m.webb locked out](../links/screenshots/splunk-queries/win-event-4740.png)

## 2. DVWA Successful Login Events

### SPL Query
```spl
index=main sourcetype=apache_access host="dvwa-server" uri_path="/DVWA/index.php" method="GET"
```

After a successful login, DVWA redirects the user to `/DVWA/index.php`. Querying for `GET` requests to this endpoint is a reliable indicator of successful authentication, since this page is only reached after valid credentials are submitted. This approach is more precise than filtering on status codes alone, which can vary depending on the client browser or tool used.

### Field Extractions

Fields such as `clientip`, `status`, `method`, and `uri_path` are not automatically extracted from Apache access logs in Splunk. An equivalent query without extracted fields might look like:

```spl
index=main sourcetype=apache_access host="dvwa-server" "/DVWA/index.php" "GET"
```

Custom field extractions were created via the Field Extractor wizard (**Settings → Fields → Field Extractions**) by highlighting values in a sample event and naming them according to Splunk CIM conventions.

![Field extraction wizard](../links/screenshots/splunk-queries/field-extraction.png)

### Result

The query returned successful login events to DVWA, showing the source IPs and timestamps of each authenticated session. This is useful for identifying who is accessing the application and from where — in particular, logins from unexpected IPs such as the Kali attacker at `10.10.10.10`.

![DVWA successful login events with extracted fields](../links/screenshots/splunk-queries/dvwa-successful-login-events-with-new-fields.png)

Note: Apache access logs only provide HTTP metadata. In production environments, dedicated auth logging via the application, an identity provider (e.g. Okta, Azure AD), or a WAF would provide more reliable login visibility.


## Sections to Cover
- SPL queries for detecting brute force, command injection, and lateral movement
- Correlation searches linking events across endpoints
- Alert dashboard configuration and thresholds
- Example alerts triggered by attack exercises
- Windows Alert Analysis
- Wireshark Packet Analysis (Windows Event Forwarding to Domain Controller)
