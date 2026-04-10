# Lab 01: Suspicious PowerShell Execution

## Overview
This lab demonstrates how suspicious PowerShell execution generates endpoint telemetry and how a SOC analyst would identify and triage that activity using Sysmon.

The scenario simulates attacker tradecraft, not malware execution. All activity is performed safely within an isolated lab environment.

---

## Lab Objectives
- Generate endpoint telemetry using suspicious PowerShell execution
- Identify relevant Sysmon events
- Analyze command-line behavior from a SOC perspective
- Understand how legitimate tools can resemble malicious activity
- Practice SOC-style reasoning and documentation

---

## Environment
- **Operating System:** Windows 10 Pro (VirtualBox VM)
- **Telemetry Tool:** Sysmon (SwiftOnSecurity configuration)
- **Network:** Host-only (isolated)
- **User Context:** Local administrator

---

## Scenario Description
PowerShell is frequently abused by attackers because it is a native Windows tool capable of executing commands, scripts, and payloads without requiring additional binaries.

In this lab, PowerShell is executed with flags commonly associated with defense evasion and stealth, allowing us to observe how this behavior appears in endpoint telemetry.

---

## Step 1: Telemetry Generation

### Action Performed
PowerShell was executed with the following command:
- powershell -nop -w hidden -c "Get-Process."
<img width="906" height="176" alt="lab01-powershell-command" src="https://github.com/user-attachments/assets/fc24c4cd-a999-4668-948c-514bb174b4ed" />

## Step 2: Telemetry Identification

### Log Source 
The activity was captured by Sysmon and identified under:
Event Viewer ->
Applications and Services Logs ->
Microsoft ->
Windows ->
Sysmon ->
Operational 

### Relevant Event
- Sysmon Event ID: 1 (Process Creation)

## Step 3: Event Analysis

### The following fields were reviewed in the Sysmon event:
- Image: powershell.exe
- CommandLine: Contains -nop -w hidden -c
- ParentImage: powershell.exe
- User: DESKTOP-U2QKAD9\soclab
- IntegrityLevel: High

The presence of hidden execution and profile suppression indicates behavior commonly associated with threat actor activity and defense evasion techniques.

## SOC Analysis

### Why is PowerShell running with a hidden window?

PowerShell was executed with the -w hidden flag to suppress visible execution, a technique commonly used to evade user awareness and increase stealth.

### Why would an attacker use -nop?

The -nop (NoProfile) flag prevents PowerShell from loading user or system profiles, helping attackers evade logging, security controls, and defensive scripts configured in profiles.

### Would this behavior be normal for a user workstation?

No. Hidden PowerShell execution with profile suppression is uncommon for normal user activity and may indicate automation or malicious intent.

### What legitimate scenarios could exist?
Legitimate scenarios may include endpoint management tools, IT automation scripts, or software deployment agents that execute PowerShell silently. In enterprise environments, such activity should be documented, expected, and associated with approved tooling or scheduled tasks.

## Evidence
- Screenshot: Sysmon Event ID 1 showing suspicious PowerShell command-line execution
- <img width="1238" height="622" alt="lab01-sysmon-eventid1" src="https://github.com/user-attachments/assets/e5800316-2d4a-43a7-99ca-d47134f4b7b6" />

## SOC Triage Considerations
- If observed in a production environment, a SOC analyst would:
-   Verify whether the activity is expected (approved scripts or management tools)
-   Correlate with user behavior and host history
-   Review additional telemetry (network connections, child processes)
-   Escalate if the activity is unauthorized or unexplained.

## Conclusion
This lab demonstrates how attacker-like behavior can be generated safely and identified through endpoint telemetry. It reinforces the importance of analyzing behavior, not just payloads, and highlights how attackers can abuse legitimate tools.

## MITRE ATT&CK mapping
- T1059.001 - Command and Scripting Interpreter: PowerShell



