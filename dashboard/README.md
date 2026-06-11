# Homelab SOC Dashboard

## Overview

This Splunk Dashboard Studio dashboard provides real-time visibility across two victim hosts in an isolated homelab environment — a Windows 11 VM and an Ubuntu 24.04 VM. It was built as part of a structured attack simulation project to practice blue team detection and incident response skills. The dashboard surfaces authentication failures, event volume trends, and top source IPs, giving a SOC analyst an immediate picture of activity across the environment.

---

## Hosts Monitored

| Host | OS | Log Sources |
|---|---|---|
| `DESKTOP-6P0RIHQ` | Windows 11 | WinEventLog:Security, WinEventLog:System, XmlWinEventLog:Microsoft-Windows-Sysmon/Operational |
| `ubuntu-VMware-Virtual-Platform` | Ubuntu 24.04 | /var/log/syslog, /var/log/auth.log |

---

![Dashboard Overview](https://github.com/Latte4Lab/homelab-SOC-project/blob/main/dashboard/screenshots/Pannel%201%20to%20%203.png)
![Dashboard Overview](https://github.com/Latte4Lab/homelab-SOC-project/blob/main/dashboard/screenshots/Pannel%204%20to%205.png)

---
## Dashboard Panels

### 1. Total Events Over Time
Tracks overall log volume across all hosts over time. Spikes indicate bursts of activity, useful for correlating with known attack simulation timestamps.
```spl
index=main | timechart span=1h count by host
```

### 2. Events by Host
Shows the distribution of log events between the Windows and Ubuntu VMs. Quick health check of which host is generating the most activity.
```spl
index=main | stats count by host
```

### 3. Events by Log Type
Breaks down ingested events by sourcetype. Confirms all log sources are actively forwarding.
```spl
index=main | stats count by sourcetype | sort -count
```

### 4. Failed Authentication Attempts
Tracks failed login attempts over time from both hosts. Brute force, SSH banner grabbing, and credential attacks all will create spikes here.
```spl
index=main (host="ubuntu*" OR host="DESKTOP*") (source="/var/log/auth.log" OR sourcetype="WinEventLog:Security") | timechart span=15m count
```

### 5. Top Source IPs
Identifies the most active source IPs. During attack simulations the Kali attacker IP (192.168.80.129) surfaces here.
```spl
index=main
| rex field=_raw "(?i)(?:from|src|source[\s_]?ip|rhost|client)[\s=:]+(?P<src_ip>\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})"
| where isnotnull(src_ip)
| where src_ip!="127.0.0.1" AND src_ip!="0.0.0.0" AND src_ip!="192.168.80.1"
| stats count by src_ip
| sort -count
```

---

## Lab Network

| Host | IP |
|---|---|
| Kali Linux (Attacker) | 192.168.80.129 |
| Windows 11 (Victim 1) | 192.168.80.128 |
| Ubuntu 24.04 (Victim 2) | 192.168.80.130 |
| Host PC (Splunk) | 192.168.80.1 |

