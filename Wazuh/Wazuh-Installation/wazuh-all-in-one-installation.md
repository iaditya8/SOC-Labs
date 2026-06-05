# Wazuh All-in-One Installation Guide

## Ubuntu Server 22.04.5 LTS

---

# Project Information

| Field            | Value                         |
| ---------------- | ----------------------------- |
| Project ID       | WAZUH-INSTALL-001             |
| Title            | Wazuh All-in-One Installation |
| Status           | Completed                     |
| Difficulty       | Beginner                      |
| Platform         | VMware Workstation Pro        |
| Operating System | Ubuntu Server 22.04.5 LTS     |

---

# Objective

Deploy a complete Wazuh SIEM environment using the All-in-One architecture.

The installation includes:

* Wazuh Dashboard
* Wazuh Indexer
* Wazuh Manager
* Wazuh API

All components run on a single Ubuntu Server.

---

# Lab Environment

## Hypervisor

VMware Workstation Pro

## Virtual Machine Specifications

| Resource | Value                     |
| -------- | ------------------------- |
| RAM      | 8 GB+                     |
| CPU      | 2+ vCPUs                  |
| Disk     | 50 GB                     |
| Network  | NAT                       |
| OS       | Ubuntu Server 22.04.5 LTS |

---

# Architecture

```text
Windows Browser
       |
       |
       v
Wazuh Dashboard
       |
       |
       v
Wazuh API
       |
       |
       v
Wazuh Manager
       |
       |
       v
Wazuh Indexer
```

---

# Step 1 - Install Ubuntu Server

Download:

Ubuntu Server 22.04.5 LTS

Create VM in VMware Workstation Pro.

Recommended settings:

```text
RAM: 8 GB
CPU: 2 vCPU
Disk: 50 GB
Network: NAT
```

Install Ubuntu using default installation options.

---

# Step 2 - Update System

Command:

```bash
sudo apt update
```

Purpose:

Refresh package repository information.

Explanation:

```text
sudo   -> Run command as administrator
apt    -> Package manager
update -> Download package metadata
```

---

Command:

```bash
sudo apt upgrade -y
```

Purpose:

Install latest package updates.

Explanation:

```text
upgrade -> Upgrade installed packages
-y      -> Automatically answer yes
```

---

# Step 3 - Verify Network Connectivity

Check IP address:

```bash
hostname -I
```

Purpose:

Display assigned IP address.

Example:

```text
192.168.21.135
```

---

Alternative:

```bash
ip a
```

Purpose:

Display complete network configuration.

---

Verify Internet connectivity:

```bash
ping -c 4 google.com
```

Purpose:

Verify outbound connectivity.

---

# Step 4 - Download Wazuh Installation Script

Command:

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
```

Explanation:

```text
curl -> Download files from URLs
-s   -> Silent mode
-O   -> Save using original filename
```

Downloaded file:

```text
wazuh-install.sh
```

---

# Step 5 - Install Wazuh All-in-One

Command:

```bash
sudo bash wazuh-install.sh -a
```

Explanation:

```text
sudo -> Administrator privileges
bash -> Execute script
-a   -> All-in-One deployment
```

---

# Components Installed

## Wazuh Dashboard

Purpose:

Web interface used by analysts.

Functions:

* Threat Hunting
* Vulnerability Detection
* FIM
* Agent Management
* Compliance Monitoring

---

## Wazuh Manager

Purpose:

Core SIEM engine.

Functions:

* Event processing
* Rule matching
* Alert generation
* Agent communication

---

## Wazuh Indexer

Purpose:

Store and search security events.

Functions:

* Data indexing
* Search operations
* Dashboard queries

---

## Wazuh API

Purpose:

Communication layer between Dashboard and Manager.

Default Port:

```text
55000
```

---

# Step 6 - Verify Services

Check Dashboard:

```bash
sudo systemctl status wazuh-dashboard
```

Expected:

```text
active (running)
```

---

Check Manager:

```bash
sudo systemctl status wazuh-manager
```

Expected:

```text
active (running)
```

---

Check Indexer:

```bash
sudo systemctl status wazuh-indexer
```

Expected:

```text
active (running)
```

---

# Step 7 - Access Dashboard

Determine server IP:

```bash
hostname -I
```

Example:

```text
192.168.21.135
```

Open browser:

```text
https://192.168.21.135
```

Ignore certificate warning.

Proceed to dashboard.

---

# Step 8 - Obtain Default Credentials

Generated credentials are stored by the installation script.

Example commands used during troubleshooting:

```bash
sudo tar -xvf wazuh-install-files.tar
```

Review generated passwords.

Store securely.

---

# Step 9 - Verify Installed Version

Command:

```bash
dpkg -l | grep wazuh
```

Purpose:

Display installed Wazuh packages.

Example:

```text
wazuh-dashboard
wazuh-indexer
wazuh-manager
```

---

# Useful Commands Learned

## Check IP Address

```bash
hostname -I
```

---

## Check Services

```bash
sudo systemctl status wazuh-manager
```

---

## Check Listening Ports

```bash
sudo ss -tulpn
```

---

## Check Installed Packages

```bash
dpkg -l | grep wazuh
```

---

## Reboot System

```bash
sudo reboot
```

---

# Snapshot Strategy

Create VMware snapshots:

Before installation

```text
Ubuntu-Fresh-Install
```

After successful installation

```text
Wazuh-Installed
```

Before upgrades

```text
Before-Wazuh-Upgrade
```

Purpose:

Quick rollback during failures.

---

# Skills Learned

* Linux Administration
* Package Management
* Service Management
* Network Verification
* SIEM Deployment
* Wazuh Architecture
* VMware Administration

---

# Validation Checklist

| Task                 | Status    |
| -------------------- | --------- |
| Ubuntu Installed     | Completed |
| Packages Updated     | Completed |
| Wazuh Downloaded     | Completed |
| Wazuh Installed      | Completed |
| Dashboard Running    | Completed |
| Manager Running      | Completed |
| Indexer Running      | Completed |
| Dashboard Accessible | Completed |

---

# Project Outcome

```text
Project ID : WAZUH-INSTALL-001
Title      : Wazuh All-in-One Installation
Status     : COMPLETED
Result     : SUCCESS
```
