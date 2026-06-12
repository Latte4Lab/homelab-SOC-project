# Ubuntu Reconnaissance Analysis

**Target:** Ubuntu 24.04 Server (192.168.80.130)
**Attacker:** Kali Linux (192.168.80.129)
**Date:** 2026-06-12
**MITRE ATT&CK:** T1595 — Active Scanning

---

## Objective

Simulate an attacker's reconnaissance phase against the Ubuntu victim using four Nmap scan types, then analyze which scans were detected by the Splunk SIEM via host-based logging (`/var/log/auth.log`).

---

## Open Ports Discovered

| Port | Service | Version |
|---|---|---|
| 22/tcp | SSH | OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 |
| 80/tcp | HTTP | Apache httpd 2.4.58 (Ubuntu) |

---

## Scans & Detection Results

| Scan Type | Command | Layer | Detected? | Evidence |
|---|---|---|---|---|
| SYN Stealth | `nmap -sS` | TCP | ❌ No | No host-based log generated |
| Version Detection | `nmap -sV` | Application | ✅ Yes | `sshd: Connection closed by 192.168.80.129` |
| OS Detection | `nmap -O` | Network/IP | ❌ No | No host-based log generated |
| Vuln Script | `nmap --script vuln` | Application | ✅ Yes | `banner exchange: invalid format` / `no matching host key [preauth]` |

---

## Analysis

### Why -sV and --script vuln were detected
Both scans actively connect to the SSH service to interact with it (grabbing version banners, probing for vulnerabilities). These connections generate pre-authentication log entries in `auth.log`. The signatures — a connection that opens and immediately closes with no successful authentication, malformed banner exchanges, and failed key negotiations — are classic reconnaissance indicators. A legitimate user authenticates; a scanner connects and vanishes.

### Why -sS and -O evaded detection
The SYN scan operates at the TCP layer (half-open: SYN → SYN-ACK → RST) and never completes a connection that SSH or Apache would log. The OS detection scan works at the network/IP layer, fingerprinting the TCP/IP stack without engaging any application service. Neither reaches the application layer, so host-based logging is blind to them.

---

## Detection Gap

Host-based logging (`auth.log`) cannot see stealth scans (`-sS`) or OS fingerprinting (`-O`) because they never interact with application-layer services. Closing this gap requires network-level detection (Suricata, Zeek, or firewall connection logging).

---

## Detection Queries

Isolate all activity from the attacker IP:
```spl
index=main host="ubuntu*" source="/var/log/auth.log" "192.168.80.129"
```

Failed authentication / connection spike over time:
```spl
index=main host="ubuntu*" source="/var/log/auth.log" | timechart span=15m count
```

---

## Screenshots

![SYN scan output](https://github.com/Latte4Lab/homelab-SOC-project/blob/main/phase1-recon/screenshots/Ubuntu/Kali%20SYN%201.png)
*Figure 1: SYN scan (-sS) — ports 22 and 80 discovered*

![Version detection](https://github.com/Latte4Lab/homelab-SOC-project/blob/main/phase1-recon/screenshots/Ubuntu/Kali%20Version%20Detection%201.png)
*Figure 2: Version detection (-sV) — OpenSSH and Apache versions identified*

![Splunk dashboard detection](https://github.com/Latte4Lab/homelab-SOC-project/blob/main/phase1-recon/screenshots/Ubuntu/Vuln%20scan%201%20(Dashboard).png)
*Figure 3: Splunk — attacker IP 192.168.80.129 detected in Dashboard*

![Splunk detection](https://github.com/Latte4Lab/homelab-SOC-project/blob/main/phase1-recon/screenshots/Ubuntu/Vuln%20scan%202.png)
*Figure 3: Splunk — attacker IP 192.168.80.129 detected in auth.log*

---

## Outcome

**2 of 4 scans detected.** Application-layer scans were caught through SSH pre-authentication logging; stealth and OS-detection scans evaded host-based detection, identifying a network-layer visibility gap.
