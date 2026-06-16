# SOC Home Lab: Splunk SIEM, Windows Sysmon, Linux Log Forwarding, and Kali-Based Detection Testing

## Project Overview

This project documents the design and implementation of a local SOC-style home lab built to practice log collection, endpoint telemetry, Linux authentication monitoring, firewall logging, and basic detection engineering using Splunk. The environment simulates a small security monitoring setup with multiple log sources forwarding into a centralized SIEM. It includes a Windows endpoint, a Linux target server, a Kali Linux testing machine, and a dedicated Splunk server.

The lab was built in a controlled virtual environment using host-only networking for testing activity. All offensive security activity was limited to lab-owned virtual machines.

---

## Objectives

The main objectives of this lab were to:

* Deploy and configure Splunk as a centralized SIEM
* Forward Windows Event Logs into Splunk
* Install and validate Sysmon endpoint telemetry
* Forward Linux authentication and system logs into Splunk
* Generate controlled SSH and scan activity from Kali Linux
* Detect failed SSH logins, successful SSH logins, and possible port scanning
* Practice writing and validating SPL searches
* Troubleshoot common log forwarding and permissions issues
* Document a reusable SOC lab build process without exposing sensitive environment details

---

## Lab Architecture

```text
+-------------------------+
|     Windows Endpoint    |
|-------------------------|
| Windows Security Logs   |
| Windows System Logs     |
| Windows Application Logs|
| Sysmon Operational Logs |
| Splunk Universal Fwd    |
+------------+------------+
             |
             | forwards Windows telemetry
             v
+-------------------------+
|      Splunk Server      |
|-------------------------|
| Ubuntu Server           |
| Splunk Enterprise       |
| index=windows           |
| index=linux             |
+------------+------------+
             ^
             | forwards Linux logs
             |
+------------+------------+
|      Ubuntu Target      |
|-------------------------|
| OpenSSH Server          |
| auth.log                |
| syslog                  |
| UFW Firewall Logs       |
| Splunk Universal Fwd    |
+------------+------------+
             ^
             |
             | controlled lab testing
             |
+------------+------------+
|        Kali Linux       |
|-------------------------|
| SSH testing             |
| Nmap scanning           |
| Host-only network       |
+-------------------------+
```

---

## Environment Summary

| System           | Role               | Primary Function                                 |
| ---------------- | ------------------ | ------------------------------------------------ |
| Splunk Server    | SIEM               | Receives and indexes Windows and Linux logs      |
| Windows Endpoint | Monitored endpoint | Sends Windows Event Logs and Sysmon telemetry    |
| Ubuntu Target    | Linux target       | Generates SSH, auth, syslog, and firewall events |
| Kali Linux       | Testing system     | Generates controlled SSH and Nmap activity       |

---

## Tools and Technologies Used

| Tool                       | Purpose                                         |
| -------------------------- | ----------------------------------------------- |
| VirtualBox                 | Virtualization platform                         |
| Ubuntu Server              | Splunk server and Linux target operating system |
| Windows 10                 | Monitored Windows endpoint                      |
| Kali Linux                 | Controlled testing and scanning system          |
| Splunk Enterprise          | Central SIEM platform                           |
| Splunk Universal Forwarder | Log forwarding agent                            |
| Sysmon                     | Windows endpoint telemetry                      |
| UFW                        | Linux firewall logging                          |
| OpenSSH Server             | SSH authentication testing                      |
| Nmap                       | Controlled port scanning                        |
| SPL                        | Splunk Search Processing Language               |

---

## Privacy and Sanitization Notice

This repository intentionally excludes sensitive or identifying details.

The following items are not included:

* Real IP addresses
* Real usernames
* Real hostnames
* Screenshots containing system details
* Public IP information
* Credentials
* VM UUIDs
* Local file paths tied to a personal machine
* Personal account information

All environment-specific values are represented with placeholders.

---

## Placeholder Reference

| Placeholder                  | Description                                  |
| ---------------------------- | -------------------------------------------- |
| `<SPLUNK_SERVER_IP>`         | Host-only IP address of the Splunk server    |
| `<WINDOWS_ENDPOINT_IP>`      | Host-only IP address of the Windows endpoint |
| `<UBUNTU_TARGET_IP>`         | Host-only IP address of the Ubuntu target    |
| `<KALI_IP>`                  | Host-only IP address of the Kali VM          |
| `<UBUNTU_USERNAME>`          | Generic Ubuntu lab user                      |
| `<WINDOWS_USERNAME>`         | Generic Windows lab user                     |
| `<SPLUNK_VERSION>`           | Installed Splunk Enterprise version          |
| `<SPLUNK_FORWARDER_VERSION>` | Installed Splunk Universal Forwarder version |
| `<TIMEZONE>`                 | Local timezone configured in the lab         |

---

# 1. Network Design

The lab was built using VirtualBox with NAT and host-only networking.

## Splunk Server Network

| Adapter   | Mode              | Purpose                                   |
| --------- | ----------------- | ----------------------------------------- |
| Adapter 1 | NAT               | Internet access for updates and downloads |
| Adapter 2 | Host-only Adapter | Internal lab communication                |

## Windows Endpoint Network

| Adapter   | Mode              | Purpose                                   |
| --------- | ----------------- | ----------------------------------------- |
| Adapter 1 | NAT               | Internet access for updates and downloads |
| Adapter 2 | Host-only Adapter | Sends logs to Splunk                      |

## Ubuntu Target Network

| Adapter   | Mode              | Purpose                                   |
| --------- | ----------------- | ----------------------------------------- |
| Adapter 1 | NAT               | Internet access for updates and downloads |
| Adapter 2 | Host-only Adapter | Receives lab traffic and forwards logs    |

## Kali Linux Network

| Adapter   | Mode              | Purpose                                   |
| --------- | ----------------- | ----------------------------------------- |
| Adapter 1 | Host-only Adapter | Isolated testing against lab systems only |

Kali Linux was intentionally kept on host-only networking during testing to prevent accidental scanning outside the lab.

---

# 2. Splunk Server Setup

## 2.1 Create the Splunk Server VM

Recommended VM configuration:

| Setting           | Value             |
| ----------------- | ----------------- |
| Operating System  | Ubuntu Server     |
| RAM               | 4 GB minimum      |
| CPU               | 2 cores           |
| Disk              | 60 GB minimum     |
| Network Adapter 1 | NAT               |
| Network Adapter 2 | Host-only Adapter |

After installation, update the system:

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 2.2 Verify Network Interfaces

```bash
ip a
```

The Splunk server should have:

```text
NAT interface       → internet access
Host-only interface → lab communication
```

The host-only IP is referenced as:

```text
<SPLUNK_SERVER_IP>
```

---

## 2.3 Install Splunk Enterprise

Download the Linux `.deb` installer from Splunk.

Example filename:

```text
splunk-<SPLUNK_VERSION>-linux-amd64.deb
```

Install Splunk:

```bash
sudo dpkg -i splunk-<SPLUNK_VERSION>-linux-amd64.deb
```

Set ownership of the Splunk directory:

```bash
sudo chown -R <UBUNTU_USERNAME>:<UBUNTU_USERNAME> /opt/splunk
```

Start Splunk:

```bash
/opt/splunk/bin/splunk start --accept-license
```

Check Splunk status:

```bash
/opt/splunk/bin/splunk status
```

Access Splunk Web:

```text
http://<SPLUNK_SERVER_IP>:8000
```

---

## 2.4 Enable Splunk Receiving

Splunk was configured to receive forwarded data on port `9997`.

```bash
/opt/splunk/bin/splunk enable listen 9997
/opt/splunk/bin/splunk restart
```

Confirm the receiving port:

```bash
/opt/splunk/bin/splunk display listen
```

Expected output:

```text
9997
```

---

## 2.5 Create Splunk Indexes

In Splunk Web:

```text
Settings → Indexes → New Index
```

Created indexes:

| Index     | Purpose                                         |
| --------- | ----------------------------------------------- |
| `windows` | Windows Event Logs and Sysmon logs              |
| `linux`   | Linux authentication, syslog, and firewall logs |

Suggested small-lab index sizes:

| Index     | Suggested Max Size |
| --------- | -----------------: |
| `windows` |              10 GB |
| `linux`   |               5 GB |

---

# 3. Windows Endpoint Setup

## 3.1 Create the Windows Endpoint VM

Recommended VM configuration:

| Setting           | Value             |
| ----------------- | ----------------- |
| Operating System  | Windows 10        |
| RAM               | 4 GB              |
| CPU               | 2 cores           |
| Disk              | 60 GB minimum     |
| Network Adapter 1 | NAT               |
| Network Adapter 2 | Host-only Adapter |

The system was renamed using PowerShell:

```powershell
Rename-Computer -NewName "windows-endpoint" -Restart
```

Verify the hostname:

```powershell
hostname
```

---

## 3.2 Test Connectivity to Splunk

From the Windows endpoint:

```powershell
ping <SPLUNK_SERVER_IP>
```

Test the Splunk receiving port:

```powershell
Test-NetConnection <SPLUNK_SERVER_IP> -Port 9997
```

Expected result:

```text
TcpTestSucceeded : True
```

---

## 3.3 Install Splunk Universal Forwarder on Windows

The Splunk Universal Forwarder was installed on the Windows endpoint.

During installation:

```text
Deployment Server: blank
Receiving Indexer: <SPLUNK_SERVER_IP>:9997
```

Check the forwarder service:

```powershell
Get-Service SplunkForwarder
```

Expected status:

```text
Running
```

Confirm forwarding:

```powershell
& "C:\Program Files\SplunkUniversalForwarder\bin\splunk.exe" list forward-server
```

Expected output:

```text
Active forwards:
    <SPLUNK_SERVER_IP>:9997
```

---

## 3.4 Configure Windows Event Log Forwarding

Edit the Universal Forwarder input configuration:

```powershell
notepad "C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf"
```

Add the following:

```ini
[WinEventLog://Security]
disabled = 0
index = windows

[WinEventLog://System]
disabled = 0
index = windows

[WinEventLog://Application]
disabled = 0
index = windows
```

Restart the Windows VM:

```powershell
Restart-Computer
```

---

## 3.5 Verify Windows Logs in Splunk

Basic search:

```spl
index=windows
```

Verify sources:

```spl
index=windows
| stats count latest(_time) as latest_event by host source sourcetype
| convert ctime(latest_event)
| sort - latest_event
```

Expected sources:

```text
WinEventLog:Security
WinEventLog:System
WinEventLog:Application
```

---

# 4. Sysmon Setup

## 4.1 Install Sysmon

Sysmon was downloaded from Microsoft Sysinternals.

Example working directory:

```text
C:\Users\<WINDOWS_USERNAME>\Desktop\Tools
```

Expected files:

```text
Sysmon.exe
Sysmon64.exe
Sysmon64a.exe
Eula.txt
```

Install Sysmon:

```powershell
cd C:\Users\<WINDOWS_USERNAME>\Desktop\Tools
.\Sysmon64.exe -accepteula -i
```

Check the Sysmon service:

```powershell
Get-Service Sysmon64
```

Expected status:

```text
Running
```

Verify local Sysmon events:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 5
```

Useful Sysmon event IDs:

| Event ID | Description        |
| -------: | ------------------ |
|        1 | Process Create     |
|        3 | Network Connection |
|        5 | Process Terminated |
|       11 | File Create        |
|       22 | DNS Query          |

The available event IDs may vary depending on the Sysmon configuration used.

---

## 4.2 Forward Sysmon Logs to Splunk

Edit the Windows forwarder input configuration:

```powershell
notepad "C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf"
```

Final Windows input configuration:

```ini
[WinEventLog://Security]
disabled = 0
index = windows

[WinEventLog://System]
disabled = 0
index = windows

[WinEventLog://Application]
disabled = 0
index = windows

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
index = windows
```

---

## 4.3 Troubleshoot Sysmon Forwarding Permissions

The forwarder successfully sent standard Windows logs but did not initially ingest Sysmon logs.

The Splunk Forwarder service account was checked:

```powershell
Get-CimInstance Win32_Service -Filter "Name='SplunkForwarder'" | Select-Object Name, StartName, State
```

Problem condition:

```text
StartName : NT SERVICE\SplunkForwarder
```

The service was changed to run as LocalSystem:

```powershell
sc.exe config SplunkForwarder obj= LocalSystem
```

Expected output:

```text
[SC] ChangeServiceConfig SUCCESS
```

Reboot:

```powershell
Restart-Computer
```

Confirm service account:

```powershell
Get-CimInstance Win32_Service -Filter "Name='SplunkForwarder'" | Select-Object Name, StartName, State
```

Expected output:

```text
StartName : LocalSystem
State     : Running
```

After this change, the Universal Forwarder was able to read and forward the Sysmon Operational log.

---

## 4.4 Verify Sysmon Events in Splunk

Broad Sysmon search:

```spl
index=windows source=*Sysmon*
```

Process creation search:

```spl
index=windows source=*Sysmon* EventCode=1
| table _time host User Image CommandLine ParentImage
| sort - _time
```

Generate test process events on Windows:

```powershell
notepad
calc
cmd
whoami
ipconfig
```

Search for test processes:

```spl
index=windows source=*Sysmon* EventCode=1
| search Image="*notepad.exe" OR Image="*calc.exe" OR Image="*cmd.exe" OR Image="*ipconfig.exe" OR Image="*whoami.exe"
| table _time host User Image CommandLine ParentImage
| sort - _time
```

---

# 5. Ubuntu Target Setup

## 5.1 Create the Ubuntu Target VM

Recommended VM configuration:

| Setting           | Value             |
| ----------------- | ----------------- |
| Operating System  | Ubuntu Server     |
| RAM               | 2 GB              |
| CPU               | 1–2 cores         |
| Disk              | 30 GB             |
| Network Adapter 1 | NAT               |
| Network Adapter 2 | Host-only Adapter |

During installation, OpenSSH Server was selected.

After installation:

```bash
sudo apt update && sudo apt upgrade -y
```

Confirm SSH is running:

```bash
sudo systemctl status ssh
```

Enable SSH on boot:

```bash
sudo systemctl enable ssh
```

Confirm the host-only IP:

```bash
ip a
```

The Ubuntu target IP is referenced as:

```text
<UBUNTU_TARGET_IP>
```

---

## 5.2 Test Connectivity

From the Splunk server:

```bash
ping -c 4 <UBUNTU_TARGET_IP>
```

From the Ubuntu target:

```bash
ping -c 4 <SPLUNK_SERVER_IP>
```

Test the Splunk receiving port:

```bash
nc -vz <SPLUNK_SERVER_IP> 9997
```

If `nc` is missing:

```bash
sudo apt install netcat-openbsd -y
```

Expected output:

```text
Connection to <SPLUNK_SERVER_IP> 9997 port [tcp/*] succeeded!
```

---

## 5.3 Install Splunk Universal Forwarder on Ubuntu Target

Download the Linux `.deb` Splunk Universal Forwarder package.

Example filename:

```text
splunkforwarder-<SPLUNK_FORWARDER_VERSION>-linux-amd64.deb
```

Install the package:

```bash
sudo dpkg -i splunkforwarder-<SPLUNK_FORWARDER_VERSION>-linux-amd64.deb
```

Start the forwarder:

```bash
sudo /opt/splunkforwarder/bin/splunk start --accept-license
```

Add the Splunk server as the receiving indexer:

```bash
sudo /opt/splunkforwarder/bin/splunk add forward-server <SPLUNK_SERVER_IP>:9997
```

Confirm forwarding:

```bash
sudo /opt/splunkforwarder/bin/splunk list forward-server
```

Expected output:

```text
Active forwards:
    <SPLUNK_SERVER_IP>:9997
```

---

## 5.4 Configure Linux Log Inputs

Edit the Linux forwarder input configuration:

```bash
sudo nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

Add the following:

```ini
[monitor:///var/log/auth.log]
disabled = 0
index = linux
sourcetype = linux_secure

[monitor:///var/log/syslog]
disabled = 0
index = linux
sourcetype = syslog
```

Restart the forwarder:

```bash
sudo /opt/splunkforwarder/bin/splunk restart
```

Confirm input status:

```bash
sudo /opt/splunkforwarder/bin/splunk list inputstatus
```

Expected monitored files:

```text
/var/log/auth.log
/var/log/syslog
```

---

## 5.5 Verify Linux Logs in Splunk

Basic search:

```spl
index=linux
```

Verify Linux sources:

```spl
index=linux
| stats count by host source sourcetype
```

Expected sources:

```text
/var/log/auth.log
/var/log/syslog
```

---

# 6. SSH Authentication Testing

## 6.1 Failed SSH Login Test

From another lab VM:

```bash
ssh fakeuser@<UBUNTU_TARGET_IP>
```

Enter an incorrect password multiple times.

Search in Splunk:

```spl
index=linux sourcetype=linux_secure "Failed password"
| table _time host source _raw
| sort - _time
```

Extract source IPs:

```spl
index=linux sourcetype=linux_secure "Failed password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| stats count by src_ip
| sort - count
```

---

## 6.2 Successful SSH Login Test

From another lab VM:

```bash
ssh <UBUNTU_USERNAME>@<UBUNTU_TARGET_IP>
```

Exit the session:

```bash
exit
```

Search in Splunk:

```spl
index=linux sourcetype=linux_secure "Accepted password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| table _time host src_ip _raw
| sort - _time
```

---

## 6.3 Combined SSH Activity Search

```spl
index=linux sourcetype=linux_secure ("Failed password" OR "Accepted password")
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| stats count by src_ip
| sort - count
```

This search gives a quick view of which source IPs generated SSH authentication activity.

---

# 7. Kali Linux Setup

## 7.1 Create the Kali VM

Kali Linux was added to generate controlled test activity.

Recommended settings:

| Setting           | Value              |
| ----------------- | ------------------ |
| Operating System  | Kali Linux         |
| RAM               | 2–4 GB             |
| CPU               | 2 cores            |
| Network Adapter 1 | Host-only Adapter  |
| NAT               | Disabled initially |
| Shared Clipboard  | Disabled           |
| Drag and Drop     | Disabled           |
| Shared Folders    | None               |

Kali was kept isolated on host-only networking to ensure all testing remained inside the lab.

---

## 7.2 Verify Kali Network Connectivity

On Kali:

```bash
ip a
```

The Kali IP is referenced as:

```text
<KALI_IP>
```

Ping the Ubuntu target:

```bash
ping -c 4 <UBUNTU_TARGET_IP>
```

Test SSH reachability:

```bash
nc -vz <UBUNTU_TARGET_IP> 22
```

If `nc` is missing:

```bash
sudo apt update
sudo apt install netcat-openbsd -y
```

---

## 7.3 Generate SSH Events from Kali

Failed SSH test:

```bash
ssh fakeuser@<UBUNTU_TARGET_IP>
```

Successful SSH test:

```bash
ssh <UBUNTU_USERNAME>@<UBUNTU_TARGET_IP>
```

Splunk search for failed attempts:

```spl
index=linux sourcetype=linux_secure "Failed password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| table _time host src_ip _raw
| sort - _time
```

Splunk search for successful logins:

```spl
index=linux sourcetype=linux_secure "Accepted password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| table _time host src_ip _raw
| sort - _time
```

---

# 8. UFW Firewall Logging and Nmap Detection

## 8.1 Enable UFW on Ubuntu Target

On the Ubuntu target:

```bash
sudo apt update
sudo apt install ufw -y
sudo ufw allow ssh
sudo ufw logging on
sudo ufw enable
sudo ufw status verbose
```

This allows SSH while enabling firewall logging.

---

## 8.2 Run Controlled Nmap Scans from Kali

Only scan lab-owned systems.

Basic SYN scan:

```bash
nmap -sS -Pn <UBUNTU_TARGET_IP>
```

Service/version scan:

```bash
nmap -sV <UBUNTU_TARGET_IP>
```

Optional aggressive scan:

```bash
nmap -A -Pn <UBUNTU_TARGET_IP>
```

---

## 8.3 Search UFW Logs in Splunk

Basic UFW search:

```spl
index=linux sourcetype=syslog "UFW"
| table _time host _raw
| sort - _time
```

Extract source IP and destination port:

```spl
index=linux sourcetype=syslog "UFW"
| rex "SRC=(?<src_ip>\d+\.\d+\.\d+\.\d+)"
| rex "DPT=(?<dest_port>\d+)"
| stats count by src_ip dest_port
| sort - count
```

---

## 8.4 Basic Port Scan Detection Search

This search looks for a source IP contacting multiple destination ports.

```spl
index=linux sourcetype=syslog "UFW"
| rex "SRC=(?<src_ip>\d+\.\d+\.\d+\.\d+)"
| rex "DPT=(?<dest_port>\d+)"
| stats dc(dest_port) as unique_ports count by src_ip
| where unique_ports >= 5
| sort - unique_ports
```

Example interpretation:

```text
A single source IP attempted connections to multiple destination ports on the Ubuntu target. In this lab, the activity was expected and generated from the Kali VM using Nmap.
```

---

# 9. Useful SPL Searches

## 9.1 Windows Source Validation

```spl
index=windows
| stats count latest(_time) as latest_event by host source sourcetype
| convert ctime(latest_event)
| sort - latest_event
```

---

## 9.2 Windows Successful Logons

```spl
index=windows source="WinEventLog:Security" EventCode=4624
| table _time host Account_Name Logon_Type IpAddress
| sort - _time
```

---

## 9.3 Windows Failed Logons

```spl
index=windows source="WinEventLog:Security" EventCode=4625
| table _time host Account_Name Failure_Reason IpAddress
| sort - _time
```

---

## 9.4 Sysmon Process Creation

```spl
index=windows source=*Sysmon* EventCode=1
| table _time host User Image CommandLine ParentImage
| sort - _time
```

---

## 9.5 PowerShell Process Activity

```spl
index=windows source=*Sysmon* EventCode=1 powershell
| table _time host User Image CommandLine ParentImage
| sort - _time
```

---

## 9.6 Linux Source Validation

```spl
index=linux
| stats count by host source sourcetype
```

---

## 9.7 Failed SSH Logins

```spl
index=linux sourcetype=linux_secure "Failed password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| table _time host src_ip _raw
| sort - _time
```

---

## 9.8 Successful SSH Logins

```spl
index=linux sourcetype=linux_secure "Accepted password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| table _time host src_ip _raw
| sort - _time
```

---

## 9.9 SSH Activity by Source IP

```spl
index=linux sourcetype=linux_secure ("Failed password" OR "Accepted password")
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| stats count by src_ip
| sort - count
```

---

## 9.10 UFW Firewall Events

```spl
index=linux sourcetype=syslog "UFW"
| table _time host _raw
| sort - _time
```

---

## 9.11 Possible Port Scan Activity

```spl
index=linux sourcetype=syslog "UFW"
| rex "SRC=(?<src_ip>\d+\.\d+\.\d+\.\d+)"
| rex "DPT=(?<dest_port>\d+)"
| stats dc(dest_port) as unique_ports count by src_ip
| where unique_ports >= 5
| sort - unique_ports
```

---

## 9.12 Recent Events by Index Time

Useful if event timestamps appear incorrect.

```spl
index=windows OR index=linux
| where _indextime > relative_time(now(), "-15m")
| eval indextime=strftime(_indextime, "%m/%d/%Y %I:%M:%S %p")
| table indextime _time host index source sourcetype _raw
| sort - indextime
```

---

# 10. Troubleshooting Notes

## 10.1 Splunk Web Not Reachable from Host Browser

Issue:

```text
<NAT_IP>:8000 timed out
```

Cause:

The VM NAT address was not directly reachable from the host browser.

Fix:

Use a host-only adapter and browse to:

```text
http://<SPLUNK_SERVER_IP>:8000
```

---

## 10.2 Windows Forwarder Connected but Sysmon Logs Missing

Issue:

```text
Active forwards:
    <SPLUNK_SERVER_IP>:9997
```

Standard Windows logs were forwarded, but Sysmon logs were missing.

Cause:

The Splunk Forwarder service was running as:

```text
NT SERVICE\SplunkForwarder
```

Fix:

```powershell
sc.exe config SplunkForwarder obj= LocalSystem
Restart-Computer
```

Confirm:

```powershell
Get-CimInstance Win32_Service -Filter "Name='SplunkForwarder'" | Select-Object Name, StartName, State
```

Expected:

```text
StartName : LocalSystem
State     : Running
```

---

## 10.3 Incorrect Sysmon Log Name

Incorrect:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operations"
```

Correct:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational"
```

---

## 10.4 Splunk Time or Event Time Appears Incorrect

Windows time checks:

```powershell
Get-Date
Get-TimeZone
Set-TimeZone -Id "<WINDOWS_TIMEZONE_ID>"
w32tm /resync
```

Ubuntu time checks:

```bash
timedatectl
sudo timedatectl set-timezone <TIMEZONE>
```

Splunk Web timezone setting:

```text
Top-right username → Account Settings → Time zone → <TIMEZONE>
```

---

## 10.5 Linux Forwarder Installed but Logs Missing

Check forwarding:

```bash
sudo /opt/splunkforwarder/bin/splunk list forward-server
```

Check monitored inputs:

```bash
sudo /opt/splunkforwarder/bin/splunk list inputstatus
```

Check forwarder logs:

```bash
sudo tail -n 50 /opt/splunkforwarder/var/log/splunk/splunkd.log
```

Review input configuration:

```bash
sudo cat /opt/splunkforwarder/etc/system/local/inputs.conf
```

Expected:

```ini
[monitor:///var/log/auth.log]
disabled = 0
index = linux
sourcetype = linux_secure

[monitor:///var/log/syslog]
disabled = 0
index = linux
sourcetype = syslog
```

Restart the forwarder:

```bash
sudo /opt/splunkforwarder/bin/splunk restart
```

---

# 11. Snapshot Strategy

Snapshots were taken after each major working phase.

Recommended snapshot names:

| VM               | Snapshot Name                                    |
| ---------------- | ------------------------------------------------ |
| Splunk Server    | `splunk-server-receiving-windows-and-linux-logs` |
| Windows Endpoint | `windows-endpoint-sysmon-forwarding-working`     |
| Ubuntu Target    | `ubuntu-target-linux-and-ufw-logs-forwarding`    |
| Kali VM          | `kali-host-only-testing-ready`                   |

Snapshots provide rollback points if later changes break networking, forwarding, or log ingestion.

---

# 12. Current Lab Status

At this stage, the lab successfully collects:

| Source                    | Destination Index |
| ------------------------- | ----------------- |
| Windows Security logs     | `index=windows`   |
| Windows System logs       | `index=windows`   |
| Windows Application logs  | `index=windows`   |
| Sysmon Operational logs   | `index=windows`   |
| Linux authentication logs | `index=linux`     |
| Linux syslog              | `index=linux`     |
| UFW firewall logs         | `index=linux`     |

Validated activity:

* Windows Event Logs forwarding into Splunk
* Sysmon process creation visible in Splunk
* Linux authentication logs visible in Splunk
* Failed SSH attempts visible in Splunk
* Successful SSH logins visible in Splunk
* Kali-generated Nmap scan activity visible through UFW/syslog
* Basic SPL detection for possible port scanning

---

# 13. Skills Demonstrated

This lab demonstrates hands-on experience with:

* SIEM deployment and configuration
* Splunk Enterprise administration basics
* Splunk Universal Forwarder setup
* Windows Event Log ingestion
* Sysmon deployment and troubleshooting
* Linux log forwarding
* SSH authentication log analysis
* Firewall log analysis
* Nmap scan detection
* SPL query writing
* Log source validation
* Endpoint telemetry analysis
* Safe lab isolation practices
* Troubleshooting network and forwarding issues
