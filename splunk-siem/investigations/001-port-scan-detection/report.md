# INV-001: Port Scan Detection via Nmap

**Date:** 10 March 2026  
**Analyst:** Béla Bertalan  
**Severity:** Low  
**Status:** Closed - Simulated lab exercise  
**MITRE ATT&CK Tactic:** TA0043 - Reconnaissance (same subnet)<br>
(the use of nmap -sV for service and version enumeration also aligns with T1595.002 (Active Scanning: Vulnerability Scanning))<br>
**MITRE ATT&CK Technique:** T1046 - Network Service Discovery

---

## Summary

A network port scan was detected originating from the attack machine (Kali Linux, 192.168.106.128) targeting the Windows 10 host (192.168.106.129). The scan was identified through a burst of Windows Filtering Platform connection events (EventCode 5156) logged within a two-second window, indicating automated scanning behaviour rather than normal user traffic. Three open ports were identified by the scanner: 445 (SMB), 135 (RPC), and 139 (NetBIOS).

---

## Environment

| Role           | OS                                           | IP Address      |
| -------------- | -------------------------------------------- | --------------- |
| Attack Machine | Kali Linux 2025.4                            | 192.168.106.128 |
| Target Machine | Windows 10 Pro                               | 192.168.106.129 |
| SIEM           | Ubuntu Server 24.04 + Splunk Enterprise 10.2 | 192.168.106.130 |

All machines connected on VMware VMnet1 (Host-only, isolated network).

---

## Attack Simulation

From the Kali Linux terminal, a service version scan was executed against the Windows 10 target:

```bash
nmap -sV 192.168.106.129
```

The `-sV` flag instructs Nmap to probe open ports and attempt to identify the running service and version. This is a standard reconnaissance technique used by attackers to map a target's attack surface before choosing an exploitation path.

**Nmap output:**

```
Host is up (0.00041s latency).
Not shown: 997 closed tcp ports (reset)
PORT    STATE SERVICE       VERSION
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
```

_Screenshot: kali-nmap-output.png_

> **Note:** Windows Defender Firewall was temporarily disabled on the target VM to allow the scan to return results. Audit policy was enabled on the target VM to ensure connection events were logged:
>
> ```
> auditpol /set /subcategory:"Filtering Platform Connection" /success:enable /failure:enable
> ```

---

## Detection

Splunk was used to search for Windows Filtering Platform connection events generated during the scan window. The following SPL query was used to identify and summarise the scanning activity:

```spl
index=main EventCode=5156 Source_Address="192.168.106.128"
| stats count by Source_Address, Destination_Address, Destination_Port
| sort -count
```

**Results:**

| Source Address  | Destination Address | Destination Port | Count |
| --------------- | ------------------- | ---------------- | ----- |
| 192.168.106.128 | 192.168.106.129     | 445              | 31    |
| 192.168.106.128 | 192.168.106.129     | 135              | 3     |
| 192.168.106.128 | 192.168.106.129     | 139              | 3     |

_Screenshot: splunk-port-scan-detection-table.png_

---

## Log Analysis

A total of 37 inbound connection events (EventCode 5156) were logged from 192.168.106.128 within approximately two seconds at 20:04:04 UTC. The key indicators of compromise (IOCs) are:

- **High event volume in short timeframe** - 37 connection attempts within 2 seconds from a single source IP is anomalous and inconsistent with normal user behaviour
- **Multiple destination ports probed** - connections targeting ports 445, 135, and 139 indicate port scanning rather than a specific application communicating
- **Source process: System (PID 4)** - the connections were handled by the Windows kernel, confirming these are inbound external probes rather than locally initiated traffic
- **Consistent source IP** - all 37 events originate from the same source (192.168.106.128), pointing to a single scanning host

Sample raw event:

```
EventCode:    5156
Direction:    Inbound
Source Address:      192.168.106.128
Source Port:         961
Destination Address: 192.168.106.129
Destination Port:    445
Protocol:            6 (TCP)
Application Name:    System
```

_Screenshot: splunk-5156-raw-event.png_

The three ports discovered are noteworthy from a threat perspective:

- **Port 445 (SMB)** - historically targeted by exploits such as EternalBlue (MS17-010), used in the WannaCry and NotPetya attacks. Discovery of this port open warrants close attention.
- **Port 135 (RPC)** - used for Windows remote procedure calls; exploitable in several known CVEs
- **Port 139 (NetBIOS)** - legacy Windows networking protocol; can expose machine names and share information to an attacker

---

## Response Actions

This was a controlled simulation in an isolated lab environment. In a real SOC environment, the appropriate response actions would be:

1. **Confirm the source IP** - verify whether 192.168.106.128 belongs to an authorised internal host or is unexpected on the network
2. **Check for follow-on activity** - search for subsequent events from the same source IP that might indicate exploitation attempts following reconnaissance (e.g. EventCode 4625 failed logins, SMB connection attempts)
3. **Review firewall rules** - confirm whether ports 445, 135 and 139 should be accessible from the source subnet
4. **Document and monitor** - if the source is internal and authorised (e.g. a vulnerability scanner), document it and continue monitoring. If unknown or external, escalate immediately

---

## Escalation Decision

**Would not escalate in isolation** - a single port scan from an internal IP in a known subnet is low severity on its own. However, this event would be flagged for monitoring and correlated with any subsequent activity from the same source. If follow-on exploitation attempts were detected (failed logins, SMB enumeration, lateral movement), this would be escalated to Tier 2 immediately.

---

## Lessons Learned

Windows does not log network connection events by default - the Filtering Platform Connection audit policy must be explicitly enabled to capture EventCode 5156 and 5157. This is an important configuration step when onboarding a new Windows endpoint into a SOC environment. Without it, port scans and inbound connection attempts would go undetected in the SIEM regardless of how well Splunk is configured. In a production SOC, verifying audit policy configuration should be part of the endpoint onboarding checklist.

---

## Appendix

| Item             | Value                                                        |
| ---------------- | ------------------------------------------------------------ |
| Detection query  | `index=main EventCode=5156 Source_Address="192.168.106.128"` |
| Key EventCode    | 5156 - WFP permitted connection                              |
| MITRE Tactic     | TA0043 Reconnaissance                                        |
| MITRE Technique  | T1046 Network Service Discovery                              |
| Attacker IP      | 192.168.106.128                                              |
| Target IP        | 192.168.106.129                                              |
| Ports Discovered | 135, 139, 445                                                |
| Tool Used        | Nmap 7.95 with -sV flag                                      |
