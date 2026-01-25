# Lab 02: Persistence via Registry Run Keys

## Overview
This lab demonstrates how attackers can establish persistence on a Windows system using registry Run keys and how this activity generates endpoint telemetry observable by a SOC analyst.

The scenario simulates attacker tradecraft in a controlled, isolated lab environment. No malware is used.

---

## Lab Objectives
- Simulate persistence using registry Run keys
- Identify registry modification telemetry using Sysmon
- Analyze persistence behavior from a SOC perspective
- Distinguish between legitimate and malicious registry activity
- Practice SOC-style documentation and reasoning

---

## Environment
- **Operating System:** Windows 10 Pro (VirtualBox VM)
- **Telemetry Tool:** Sysmon (SwiftOnSecurity configuration)
- **Network:** Host-only (isolated)
- **User Context:** Local administrator

---

## Scenario Description
Registry Run keys are a common persistence mechanism used by attackers to execute code automatically when a user logs in. Because these keys are legitimate Windows features, they are frequently abused to maintain access while blending in with normal system behavior.

In this lab, a Run key is created under the current user hive (`HKCU`) to execute PowerShell with stealth-related flags at logon.

---

## Step 1: Persistence Creation

### Action Performed
The following registry value was added using PowerShell:

  reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" 
  /v UpdateService 
  /t REG_SZ 
  /d "powershell.exe -nop -w hidden -c Get-Process" 
  /f

## Step 2: Telemetry Identification
### Log Source - The registry modification was captured by Sysmon and identified under:
Event Viewer →
Applications and Services Logs →
Microsoft →
Windows →
Sysmon →
Operational

## Relevant Event
- Sysmon Event ID: 13 (Registry Value Set)
- The event shows a new value being written to a registry Run key.

## Step 3 Event Analysis
### The following fields were reviewed in the Sysmon event:

- Event ID: 13
- TargetObject: HKCU\Software\Microsoft\Windows\CurrentVersion\Run\UpdateService
- Details: Command to execute at user logon
- Process: powershell.exe
- User: Logged-in user account
The telemetry confirms successful creation of a persistence mechanism.

## SOC Analysis
### Why are Run keys valuable to attackers?

- Run keys allow attackers to automatically execute code when a user logs in, providing reliable persistence without requiring additional exploits. Because Run keys are a legitimate Windows feature, malicious entries can blend in with normal system behavior.

### Why is HKCU often preferred over HKLM?

- The HKCU hive does not require administrative privileges to modify, making it easier for attackers to establish persistence without triggering privilege escalation alerts. Persistence under HKCU also targets a specific user, reducing visibility compared to system-wide keys.

### Why use PowerShell in a Run key?

- PowerShell is a built-in Windows tool capable of executing complex commands and scripts without additional binaries. Using PowerShell enables fileless execution and allows attackers to leverage stealth flags such as -nop and -w, hidden to evade detection.

### Would this be normal behavior on a user endpoint?

- In most environments, manually adding new Run keys that execute PowerShell is uncommon for standard users. While not inherently malicious, this behavior would warrant investigation in a SOC due to its association with persistence and defense evasion techniques.

## What enterprise tools might legitimately use Run keys?

### Legitimate use cases may include:

- Endpoint management and monitoring agents
- Software updaters
- Backup or synchronization tools
- Remote management utilities
- In enterprise environments, these entries should be documented, approved, and consistent across systems.

## How would one detect this in a SIEM?

### Detection strategies may include:

- Monitoring Sysmon Event ID 13 for changes to registry Run keys
- Alerting on new or unusual values under Run paths
- Correlating registry modifications with suspicious parent processes, such as PowerShell
- Comparing observed entries against a baseline of approved startup items

## Evidence

Screenshot: Registry Run key creation command
Screenshot: Sysmon Event ID 13 showing registry value modification

## SOC Triage Considerations

### If observed in a production environment, a SOC analyst would:

- Validate whether the Run key is associated with approved software
- Review the command stored in the registry value
- Correlate with other endpoint activity (process execution, network connections)
- Escalate if the persistence is unauthorized or unexplained

## Cleanup

### The persistence mechanism was removed after analysis using the following command:

reg delete "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v UpdateService /f

## Conclusion

This lab demonstrates how registry Run keys can be abused for persistence and how such activity is observable through endpoint telemetry. It highlights the importance of monitoring registry changes and understanding how legitimate Windows features can be leveraged by attackers.

## MITRE ATT&CK Mapping

T1547.001 – Boot or Logon Autostart Execution: Registry Run Keys / Startup Folder
