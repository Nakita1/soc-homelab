# SOC & Blue Team Labs

This directory contains hands-on security labs designed to demonstrate SOC analyst and Blue Team skills through controlled, isolated simulations.

Each lab focuses on **behavioral analysis**, **endpoint telemetry**, and **defensive investigation techniques** using native Windows tooling and Sysmon. No live malware is used; all activity is generated safely to emulate attacker tradecraft.

---

## Lab Index

### Lab 01 – Suspicious PowerShell Execution
**Focus:** Command-line analysis, defense-evasion indicators, and Sysmon process-creation telemetry.

- Simulates attacker-like PowerShell execution using hidden windows and profile suppression
- Captures and analyzes Sysmon Event ID 1 (Process Creation)
- Demonstrates SOC-style triage, reasoning, and MITRE ATT&CK mapping

**Techniques Observed:**
- Living-off-the-Land (LOLBins)
- Defense Evasion
- Command and Scripting Interpreter (PowerShell)

**MITRE ATT&CK:** T1059.001

---

## Lab Environment (Common Across Labs)

- **Host OS:** Windows 11
- **Guest OS:** Windows 10 Pro (VirtualBox VM)
- **Telemetry:** Sysmon (SwiftOnSecurity configuration)
- **Network:** Host-only (isolated)
- **Purpose:** Safe generation and analysis of security telemetry

---

## Disclaimer

These labs are for **defensive security education only**.  
They are designed to help understand attacker behavior, detection opportunities, and SOC workflows in a controlled environment.

No malicious payloads or live malware samples are included.

---

## Upcoming Labs

Planned future labs include:
- Registry persistence (Run keys)
- Suspicious parent/child process relationships
- Command-line obfuscation
- Scheduled task persistence
- SIEM ingestion and correlation

