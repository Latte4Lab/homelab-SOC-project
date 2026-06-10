# 🛡️ Homelab Cybersecurity Detection Project

A hands-on cybersecurity homelab simulating real-world attack scenarios and detecting them using a self-built SIEM pipeline. Built as a portfolio project targeting a blue team / SOC analyst role in 2026.

---

## 🏗️ Lab Architecture

Kali Linux (Attacker) — 192.168.80.129

Windows 11 VM (Victim 1) — 192.168.80.128

Ubuntu 24.04 VM (Victim 2) — 192.168.80.130

Host PC / Splunk SIEM — 192.168.80.1

All machines run on an isolated VMware VMnet1 Host-Only network. No attack traffic reaches the real internet. Logs ship from victim VMs to Splunk Enterprise via Universal Forwarder on TCP port 9997.

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|---|---|
| VMware Workstation Pro | Hypervisor for running VMs |
| Kali Linux | Attacker machine |
| Windows 11 | Victim machine 1 |
| Ubuntu 24.04 | Victim machine 2 |
| Splunk Enterprise 10.4 | SIEM — log collection and analysis |
| Splunk Universal Forwarder | Log shipping from victim VMs |
| Sysmon (olafhartong config) | Rich endpoint telemetry on Windows |
| Nmap | Network reconnaissance |
| Metasploit | Exploitation framework (Phase 2) |

---

## 📁 Project Structure

| Folder | Contents |
|---|---|
| phase1-recon/ | Nmap attack notes and screenshots |
| phase2-exploitation/ | Metasploit attack notes and screenshots |
| phase3-persistence/ | Persistence attack notes and screenshots |
| dashboard/ | Splunk dashboard documentation and screenshots |
| detection-queries/ | All SPL queries used across the project |
| IR-report/ | Final professional incident response report |

---

## 🔴 Attack Phases

### ✅ Phase 1 — Reconnaissance
- Tool: Nmap
- Target: Ubuntu VM (192.168.80.130)
- Detected via: /var/log/auth.log spike in Splunk
- MITRE ATT&CK: T1595 — Active Scanning

### 🔄 Phase 2 — Initial Access (In Progress)
- Tool: Metasploit
- Goal: Establish reverse shell on victim VM
- Detection target: Sysmon EventID 1 and EventID 3

### 🔲 Phase 3 — Persistence
- Goal: Create backdoor admin user via shell
- Detection target: Security EventID 4720 and 4732

---

## 📊 SOC Dashboard

Built a real-time monitoring dashboard in Splunk Dashboard Studio with 5 panels:

| Panel | Visualization | Purpose |
|---|---|---|
| Total Events Over Time | Bar Chart | Monitor overall log volume |
| Events by Host | Donut Chart | Log distribution across VMs |
| Events by Log Type | Table | Logging pipeline health |
| Failed Authentication Attempts | Line Chart | Detect scans and brute force |
| Top Source IPs | Table | Identify suspicious IPs |

---

## 🔍 Key Detections So Far

| Attack | Detection Method | Evidence |
|---|---|---|
| Nmap Port Scan | auth.log spike in Splunk | 9 events from 192.168.80.129 within 1 second |

---

## 📚 Resources

- MITRE ATT&CK Framework: https://attack.mitre.org
- Splunk Documentation: https://docs.splunk.com
- Sysmon olafhartong config: https://github.com/olafhartong/sysmon-modular

---

## 👤 Author

Putu Elang — Aspiring SOC Analyst / Blue Team
Building in public | 2026 job target
