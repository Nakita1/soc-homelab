# soc-homelab
VirtualBox-based SOC lab featuring an isolated Windows 10 endpoint instrumented with Sysmon to support endpoint detection, alert triage, and incident response workflows. 

# SOC & Blue Team Home Lab

## Overview
This repository documents the creation of an **isolated SOC and Blue Team home lab** designed to generate, observe, and analyze endpoint telemetry for detection engineering and incident response practice.

The lab is built with a strong emphasis on **safety, isolation, and defensive security principles**. No live malware samples are included, and all activity is performed in a controlled virtual environment.

---

## Objectives
- Build a **safe and isolated security lab** using virtualization
- Generate **high-fidelity endpoint telemetry** with Sysmon
- Practice **SOC analyst workflows**, including:
  - Process execution analysis
  - DNS activity review
  - Identifying suspicious behavior
- Document setup decisions and validation steps in a **repeatable, professional manner**

---

## Lab Architecture

### Host System
- Windows 11

### Virtualization
- Oracle VirtualBox

### Virtual Machines
- **Windows 10 Pro**
  - Role: Endpoint / Victim system
  - Instrumentation: Sysmon (Sysinternals)
  - Purpose: Generate endpoint telemetry for analysis

Additional labs (OSINT, governance/compliance) are maintained separately and are not part of this repository.

---

## Network Design & Safety

The lab is intentionally **isolated by design** to prevent unintended exposure.

- **Host-only networking**
  - No internet access during normal operation
  - VM-to-VM communication only
- **No shared folders**
- **Clipboard and drag-and-drop disabled**
- **No USB passthrough**

Internet access is enabled **temporarily and deliberately** only to download trusted tools, then immediately removed.

This approach reflects real-world defensive lab best practices.

---

## Endpoint Telemetry

### Sysmon
- Tool: Microsoft Sysinternals Sysmon
- Configuration: SwiftOnSecurity Sysmon configuration
- Purpose:
  - Capture high-value security events with reduced noise
  - Support detection and investigation workflows

### Key Event Types Collected
- **Event ID 1** – Process Creation  
- **Event ID 3** – Network Connections  
- **Event ID 22** – DNS Queries  
- File, registry, and persistence-related activity

Events are reviewed locally via **Event Viewer** and will later be forwarded to a SIEM for centralized analysis.

---

## Validation
Sysmon functionality was validated by:
- Confirming the `sysmon64` service is running
- Verifying events under:
  Event Viewer ->
  Applications and Services Logs ->
  Microsoft ->
  Windows ->
  Sysmon ->
  Operational
- Screenshots are included to demonstrate successful telemetry generation. 
