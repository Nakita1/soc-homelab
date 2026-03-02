# Lab 03: Wazuh SIEM Deployment

## Overview

This lab documents the deployment and validation of a fully functional Security Information and Event Management (SIEM) solution using Wazuh in a virtualized lab environment.

The objective was to build a foundational SOC monitoring stack capable of:

- Log ingestion
- Endpoint monitoring
- File Integrity Monitoring (FIM)
- Port activity detection
- Compliance auditing
- MITRE ATT&CK mapping
- Real-world troubleshooting and recovery

All infrastructure was deployed in an isolated lab network.

---

## Lab Architecture

| System | Role |
|--------|------|
| Ubuntu Server 22.04 LTS | Wazuh Manager, Indexer (OpenSearch), Dashboard |
| Kali Linux | Monitored Endpoint (Wazuh Agent) |

Both virtual machines were deployed in VMware and configured within a private lab network segment.

---

## Technologies Used

- Wazuh v4.7.x
- OpenSearch
- Ubuntu Server 22.04 LTS
- Kali Linux
- Nmap
- OpenSSH
- Linux File Integrity Monitoring (syscheck)

---

## Deployment Process

### Wazuh Server Installation

Updated system:

```bash
sudo apt update && sudo apt upgrade -y
```

Installed Wazuh (all-in-one deployment):

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash wazuh-install.sh -a
```

Verified services:

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

---

### Agent Deployment

Installed Wazuh agent on Linux endpoint:

```bash
curl -L -O https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_<version>_amd64.deb
sudo WAZUH_MANAGER='<SIEM_SERVER_IP>' dpkg -i wazuh-agent_<version>_amd64.deb
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

Registered the agent using key-based authentication via `manage_agents`.

---

## Detection Validation

### Port Scan Simulation

Simulated TCP SYN scan:

```bash
nmap -sS <SIEM_SERVER_IP>
```

Generated alert:

- Rule ID: 533  
- Description: Listened ports status (netstat) changed  
- Severity Level: 7  

This demonstrates host-based detection of reconnaissance-related activity.

---

### File Integrity Monitoring

Created and modified monitored file:

```bash
sudo touch /etc/soclab_test_file
echo "lab" | sudo tee -a /etc/soclab_test_file
```

Wazuh detected integrity changes within monitored directories.

---

### Security Configuration Assessment (SCA)

The SCA module generated compliance findings including:

- Password policy validation
- Hashing algorithm verification
- Audit service checks
- System hardening visibility

The dashboard confirmed proper alert indexing and MITRE ATT&CK mapping.

---

## Troubleshooting and Recovery

The following issues were encountered and resolved during deployment:

- Unsupported OS version
- Duplicate agent key conflict
- Service recovery after power interruption
- FIM timing adjustments for real-time monitoring

These reflect practical SOC deployment and operational challenges.

---

## Current Capabilities

- Active endpoint monitoring
- Host-based anomaly detection
- Port state change detection
- File Integrity Monitoring
- Compliance auditing
- MITRE ATT&CK mapping
- Centralized alert visualization

---

## Next Steps

- System hardening
- Detection rule tuning
- Network IDS integration (Suricata)
- Windows endpoint integration with Sysmon
- Threat intelligence integration
- Case management workflow integration

---

## Skills Demonstrated

- SIEM deployment
- Endpoint integration
- Detection validation
- Log analysis
- Attack simulation
- Incident troubleshooting
- Service recovery

---

## Project Status

Functional SIEM deployment completed.  
Hardening and advanced detection tuning in progress.
