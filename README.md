# SOC Home Lab Portfolio

A hands-on security operations lab built to develop and demonstrate practical SOC analyst skills - from setting up a SIEM and forwarding real logs, to simulating attacks, detecting them, and documenting findings as structured investigations.

---

## About Me

I'm a career-changer transitioning from software engineering into cybersecurity, currently based in the Netherlands.

My background is three years as a junior software engineer and technical mentor at a coding bootcamp in the UK, where I spent most of my time doing exactly what a SOC analyst does - just in a different domain. I handled a high volume of support tickets, debugged live issues with students over Zoom, conducted technical entry assessments for new candidates, and helped develop the company's internal systems. That work built habits that translate directly: methodical troubleshooting, clear written and verbal communication, staying calm under pressure, and explaining technical findings to non-technical people.

On the technical side I have working knowledge of JavaScript, SQL, and React, and a solid understanding of how web applications work under the hood - including the DOM, HTTP, and vulnerabilities like XSS. That background makes reading logs and understanding attack vectors more intuitive than it might be for someone coming in without any development experience.

I'm currently studying for the CCNA, working through TryHackMe, and building this lab. My goal is a Tier 1 SOC analyst role where I can put these skills to work in a real environment.

---

## Lab Overview

This portfolio is split into two environments:

### 1. On-Premises Home Lab (Splunk)

A fully isolated virtual lab running on VMware Workstation Pro with three virtual machines connected on a private host-only network.

| VM             | OS                                           | Role                 |
| -------------- | -------------------------------------------- | -------------------- |
| Attack Machine | Kali Linux 2025.4                            | Offensive simulation |
| Target Machine | Windows 10 Pro                               | Log source / victim  |
| SIEM           | Ubuntu Server 24.04 + Splunk Enterprise 10.2 | Detection & analysis |

The Windows 10 VM forwards Security, Application, and System event logs to Splunk via the Universal Forwarder. Attacks are simulated from Kali and detected in Splunk using custom SPL queries and alert rules.

### 2. Cloud Honeypot (Microsoft Sentinel) - _In Progress_

An intentionally exposed Windows VM on Azure, open to the internet to attract real-world attack traffic. Microsoft Sentinel ingests and analyses the logs, with a geographic attack map showing live threat origins. Investigations document real attacker behaviour - not simulated.

---

## Repository Structure

```
soc-home-lab/
├── environments/
│   ├── home-lab/          # VMware lab setup and network diagram
│   └── azure-cloud/       # Azure honeypot environment (coming soon)
├── splunk-siem/
│   ├── investigations/    # Structured incident reports from home lab
│   └── detection-rules/   # SPL queries and Splunk alert configurations
├── azure-sentinel/
│   └── investigations/    # Real-world attack analysis from honeypot
└── README.md
```

---

## Investigations

### Splunk SIEM

| #   | Title                            | Tactic (MITRE ATT&CK)      | Severity |
| --- | -------------------------------- | -------------------------- | -------- |
| 001 | Port Scan Detection via Nmap     | TA0043 - Reconnaissance    | Low      |
| 002 | Brute Force Login Detection      | TA0006 - Credential Access | Medium   |
| 003 | Metasploit Attack Chain Analysis | TA0002 - Execution         | High     |

### Azure Sentinel _(coming soon)_

| #   | Title                                   | Source                      |
| --- | --------------------------------------- | --------------------------- |
| 001 | Honeypot Attack Analysis                | Real-world internet traffic |
| 002 | RDP Brute Force from Multiple Countries | Real-world internet traffic |

---

## Tools & Technologies

- **SIEM:** Splunk Enterprise 10.2, Microsoft Sentinel
- **Attack tools:** Nmap, Hydra, Metasploit Framework
- **Platforms:** VMware Workstation Pro, Microsoft Azure
- **OS:** Kali Linux, Windows 10, Ubuntu Server 24.04
- **Log sources:** Windows Event Logs (Security, Application, System)
- **Frameworks:** MITRE ATT&CK

---

## Certifications & Learning

- Cisco CCNA - _in progress_
- CompTIA Security+ - _planned_
- TryHackMe - completed XSS, network and web application rooms
- 1st place - ON2IT CTF (February 2026)

---

## Contact

Open to SOC analyst opportunities in the Netherlands.  
Connect with me on <a href="https://www.linkedin.com/in/belabertalan" target="_blank">LinkedIn</a> or reach out via GitHub.
