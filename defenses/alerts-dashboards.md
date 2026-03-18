# Alerts & Dashboards

> **Documentation In Progress**

Configuration of Splunk alerts and dashboards for real-time monitoring and threat detection across the lab environment.
Note: I will probably cover these as a group once the number is substantive enough.
1. brute force alert.
index=main host="dvwa-server" sourcetype=apache_access "&password=" | stats count by clientip | where count > 50

2. top ips hitting dvwa:
index=main host="dvwa-server" sourcetype=apache_access
| stats count by clientip
| sort -count

![top-ips-hitting-dvwa](..\links\screenshots\alerts+dashboards\top-ips-hitting-dvwa.png)

## Sections to Cover
- Alert rules and trigger conditions (e.g. account lockout, brute force thresholds, suspicious login times)
- Dashboard panels and visualisations
- Alert actions (email, webhook, log event)
- Threshold tuning to reduce false positives
