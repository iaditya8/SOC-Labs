# Windows Agent Installation and Enrollment

## Wazuh Agent 4.x on Windows 10

---

# Project Information

| Field        | Value                                     |
| ------------ | ----------------------------------------- |
| Project ID   | WAZUH-INSTALL-002                         |
| Title        | Windows Agent Installation and Enrollment |
| Status       | Completed                                 |
| Difficulty   | Beginner–Intermediate                     |
| Platform     | VMware Workstation Pro                    |
| Target Agent | Windows 10                                |
| Wazuh Server | Ubuntu Server 22.04.5 LTS                 |

---

# Objective

Install and enroll a Windows endpoint into the Wazuh SIEM environment.

This project covers:

* Windows VM creation
* VMware Tools installation
* Network verification
* Wazuh Agent installation
* Agent enrollment
* Troubleshooting enrollment issues
* Version compatibility validation
* Dashboard verification

---

# Lab Environment

## Wazuh Server

| Property   | Value                            |
| ---------- | -------------------------------- |
| OS         | Ubuntu Server 22.04.5 LTS        |
| Role       | Wazuh Server                     |
| Components | Manager, Dashboard, Indexer, API |
| IP Address | <LAB_SERVER_IP>                  |

---

## Windows Endpoint

| Property | Value              |
| -------- | ------------------ |
| OS       | Windows 10         |
| Role     | Monitored Endpoint |
| Agent    | Wazuh Agent        |
| Network  | VMware NAT         |

---

# Architecture

```text
Windows 10 Endpoint
        |
        |
        v
Wazuh Agent
        |
        |
        v
Wazuh Manager
        |
        |
        v
Dashboard
```

---

# Step 1 - Create Windows Virtual Machine

Create a new VM in VMware Workstation Pro.

Recommended configuration:

```text
RAM: 4 GB
CPU: 2 vCPU
Disk: 60 GB
Network: NAT
```

Install Windows 10 normally.

---


# Step 2 - Install VMware Tools

Purpose:

VMware Tools provides:

* Clipboard sharing
* Better drivers
* Dynamic screen resizing
* Improved network performance
* Shared folders

Installation:

```text
VM → Install VMware Tools
```

Inside Windows:

Run installer.

Reboot system after installation.

---



# Step 3 - Verify Network Connectivity

Determine Wazuh Server IP.

On Ubuntu:

```bash
hostname -I
```

Example:

```text
<LAB_SERVER_IP>
```

---

Verify connectivity from Windows:

```cmd
ping <LAB_SERVER_IP>
```

Expected:

```text
Reply from <LAB_SERVER_IP>
```

---



# Step 4 - Download Wazuh Agent

Download the Windows Wazuh Agent installer.

Install using default settings.

Installation path:

```text
C:\Program Files (x86)\ossec-agent\
```

---

# Step 5 - Verify Agent Service

Open PowerShell as Administrator.

Command:

```powershell
Get-Service WazuhSvc
```

Purpose:

Verify agent service status.

Expected:

```text
Status : Running
```

---



# Step 6 - Verify Agent Version

Command:

```powershell
(Get-Item "C:\Program Files (x86)\ossec-agent\wazuh-agent.exe").VersionInfo.ProductVersion
```

Purpose:

Verify installed version.

Observed:

```text
v4.13.1
```

---

# Step 7 - Agent Enrollment

Agent enrollment performed through Wazuh Manager.

On Ubuntu:

```bash
sudo /var/ossec/bin/manage_agents
```

Purpose:

Manage Wazuh agents.

---

# Available Options

```text
A -> Add Agent
E -> Extract Key
L -> List Agents
R -> Remove Agent
```

---

# Add Agent

Select:

```text
A
```

Enter:

```text
Agent Name
IP Address
```

Example:

```text
Name: WIN10-ENDPOINT
IP: any
```

---

# Extract Enrollment Key

Select:

```text
E
```

Choose newly created agent.

Result:

```text
Agent Key Generated
```

---

# Security Note

Do not publish enrollment keys.

Replace with:

```text
<WAZUH_AGENT_KEY_REDACTED>
```

in documentation.

---

# Step 8 - Enrollment Key Transfer Problem

Issue encountered:

Clipboard sharing failed between:

* Ubuntu
* Windows

Unable to copy enrollment key.

---

# Troubleshooting VMware Clipboard

Attempted:

* Reinstall VMware Tools
* Enable copy/paste
* Restart VMware Tools

Clipboard remained unavailable.

---

# Shared Folder Workaround

Determine shared folders:

```bash
vmware-hgfsclient
```

---

Create mount point:

```bash
sudo mkdir -p /mnt/hgfs
```

---

Mount VMware shared folders:

```bash
sudo vmhgfs-fuse .host:/ /mnt/hgfs -o allow_other
```

Purpose:

Transfer enrollment keys between systems.

---

## Screenshot



# Step 9 - Import Agent Key

On Windows:

Navigate to:

```text
C:\Program Files (x86)\ossec-agent\
```

Run:

```text
manage_agents.exe
```

Select:

```text
I
```

Paste enrollment key.

Save.

---

# Step 10 - Restart Agent

PowerShell:

```powershell
Restart-Service WazuhSvc
```

Purpose:

Apply enrollment changes.

---

# Verify Service

```powershell
Get-Service WazuhSvc
```

Expected:

```text
Running
```

---

# Enrollment Failure

Observed Error:

```text
ERROR: Incompatible version for new agent
```

---

## Screenshot

```markdown
![Version Mismatch Error](images/incompatible-version-agent.png)

Figure 6: Agent enrollment failed due to version mismatch.
```

---

# Root Cause Analysis

Installed Agent Version:

```text
4.13.1
```

Server Version:

```text
4.7.5
```

Result:

```text
Agent Version > Manager Version
```

Newer agents cannot register with older managers.

---

# Validation Commands

Verify Agent Version:

```powershell
(Get-Item "C:\Program Files (x86)\ossec-agent\wazuh-agent.exe").VersionInfo.ProductVersion
```

---

Verify Service:

```powershell
Get-Service WazuhSvc
```

---

Verify Connectivity:

```cmd
ping <LAB_SERVER_IP>
```

---

Verify Manager Reachability:

```cmd
telnet <LAB_SERVER_IP> 1514
```

(Optional)

---

# Resolution

Upgrade Wazuh Server components:

* Wazuh Manager
* Wazuh Dashboard
* Wazuh Indexer

to a compatible version.

Documented in:

```text
WAZUH-INSTALL-003
wazuh-upgrade-4.7.5-to-4.14.5.md
```

---

# Lessons Learned

1. Agent and Manager versions must be compatible.
2. Always verify versions before enrollment.
3. VMware shared folders are useful when clipboard sharing fails.
4. Service status should be verified before troubleshooting enrollment.
5. Network connectivity should be validated before agent registration.

---

# Skills Demonstrated

* Windows Administration
* VMware Administration
* Endpoint Monitoring
* Wazuh Agent Management
* Network Troubleshooting
* Service Verification
* Root Cause Analysis

---

# Validation Checklist

| Task                          | Status    |
| ----------------------------- | --------- |
| Windows VM Created            | Completed |
| VMware Tools Installed        | Completed |
| Network Connectivity Verified | Completed |
| Wazuh Agent Installed         | Completed |
| Agent Service Running         | Completed |
| Enrollment Key Generated      | Completed |
| Enrollment Attempted          | Completed |
| Version Mismatch Identified   | Completed |
| Root Cause Determined         | Completed |

---

# Project Outcome

```text
Project ID : WAZUH-INSTALL-002
Title      : Windows Agent Installation and Enrollment
Status     : COMPLETED
Result     : SUCCESSFUL INSTALLATION
             ENROLLMENT BLOCKED BY VERSION MISMATCH
             ROOT CAUSE IDENTIFIED
```
