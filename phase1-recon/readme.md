# Phase 1 — Reconnaissance

Simulated attacker reconnaissance using Nmap against both victim hosts, with detection analysis in Splunk.

## MITRE ATT&CK
- **T1595** — Active Scanning
- **T1592** — Gather Victim Host Information

## Targets
| Host | IP | Result |
|---|---|---|
| Ubuntu 24.04 | 192.168.80.130 | 2 of 4 scans detected |
| Windows 11 | 192.168.80.131 | 0 of 4 scans detected |

## Key Findings
- Application-layer scans (`-sV`, `--script vuln`) were detected on Ubuntu via SSH pre-auth logs
- Stealth/network-layer scans (`-sS`, `-O`) evaded host-based detection on both hosts
- Windows host had a complete recon blind spot — Sysmon network logging (EventID 3) is disabled by default
- A suspected Windows detection (EventID 4798/4799) was ruled out as a false positive through controlled re-testing

## Detailed Writeups
- [Ubuntu Recon Analysis](ubuntu-recon.md)
- [Windows Recon Analysis](windows-recon.md)

## Detection Gap & Next Steps
Network-layer reconnaissance is currently undetectable across the environment. Planned remediation: enable Sysmon EventID 3, deploy network IDS (Suricata/Zeek), add firewall connection logging.
