# Windows Reconnaissance Analysis

**Target:** Windows 11 (192.168.80.131 — cloned VM)
**Attacker:** Kali Linux (192.168.80.129)
**Date:** 2026-06-12
**MITRE ATT&CK:** T1595 — Active Scanning

---

## Objective

Simulate an attacker's reconnaissance phase against the Windows victim using four Nmap scan types, then analyze the Windows endpoint's detection coverage via Sysmon and Windows Security auditing in Splunk.

---

## Open Ports Discovered

| Port | Service | Version |
|---|---|---|
| 135/tcp | msrpc | Microsoft Windows RPC |
| 139/tcp | netbios-ssn | Microsoft Windows NetBIOS |
| 445/tcp | microsoft-ds | SMB |

---

## Scans & Detection Results

| Scan Type | Command | Layer | Detected? | Notes |
|---|---|---|---|---|
| SYN Stealth | `nmap -sS` | TCP | ❌ No | Sysmon network logging (EventID 3) disabled |
| Version Detection | `nmap -sV` | Application | ❌ No | Suspected 4798/4799 correlation ruled out via re-test |
| OS Detection | `nmap -O` | Network/IP | ❌ No | No OS fingerprint match, no log signature |
| Vuln Script | `nmap --script vuln` | Application | ❌ No | SMB negotiation rejected; no scan logs |

---

## Vulnerability Assessment

The vuln script scan attempted to test for legacy SMB vulnerabilities:

| Check | Result |
|---|---|
| smb-vuln-ms10-061 | Could not negotiate connection (not vulnerable) |
| samba-vuln-cve-2012-1182 | Could not negotiate connection (not vulnerable) |
| smb-vuln-ms10-054 | false (not vulnerable) |

The host rejected legacy SMB1 negotiation, indicating modern SMB hardening. **The Windows 11 target is not susceptible to these classic SMB exploits.**

---

## Analyst Note — False Positive Verification

During analysis, EventID 4798/4799 (group membership enumeration) events were observed near the scan timestamps and initially appeared to be a detection.

To validate causation, a **controlled re-scan** was performed. The second scan produced **no new enumeration events**, confirming the original events were routine OS background activity — not scan-induced.

This false positive was ruled out before being attributed as a detection. It highlights the importance of correlation testing and baselining normal activity before building detection rules. Acting on the initial correlation alone would have produced a false-positive rule.

---

## Detection Gap

The Windows endpoint has **no reliable detection for network reconnaissance**:
- Sysmon EventID 3 (network connections) is disabled by the default olafhartong config
- No network-based IDS deployed
- No Windows Firewall connection logging forwarded to Splunk

This mirrors the Ubuntu blind spot but for a different reason — Ubuntu lacked network logging entirely, while Windows has Sysmon deployed but with network logging turned off.

---

## Detection Queries

Sysmon network connection check:
```spl
index=main host="DESKTOP*" sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=3 "192.168.80.129"
```

Group enumeration false-positive test:
```spl
index=main host="DESKTOP*" (EventCode=4798 OR EventCode=4799)
```

Combined attacker IP check:
```spl
index=main host="DESKTOP*" "192.168.80.129"
```

---

## Screenshots

![SYN scan output](screenshots/windows/syn-scan.png)
*Figure 1: SYN scan (-sS) — ports 135, 139, 445 discovered*

![Vuln scan output](screenshots/windows/vuln-scan.png)
*Figure 2: Vuln script scan — SMB negotiation rejected, host not vulnerable*

![Splunk no detection](screenshots/windows/splunk-no-detection.png)
*Figure 3: Splunk — no events tied to attacker IP, confirming blind spot*

---

## Planned Remediation (Future Iteration)

1. Enable Sysmon EventID 3 network logging, scoped to listening ports (135/139/445)
2. Deploy network IDS (Suricata or Zeek) for both hosts
3. Enable Windows Firewall connection logging forwarded to Splunk
4. Build volume-based correlation rules rather than relying on single event types
5. Baseline normal background activity before alerting

---

## Outcome

**0 of 4 scans detected.** The Windows host exhibits a complete network-layer reconnaissance blind spot. The exercise validated the gap, ruled out a false positive through controlled re-testing, and confirmed the host is hardened against legacy SMB exploits. This finding defines the goal for the next iteration: closing the network visibility gap and re-running these scans to demonstrate before/after detection.
