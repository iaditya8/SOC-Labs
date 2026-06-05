# SOC Lab

A hands-on Security Operations Center (SOC) lab built using Wazuh SIEM, Ubuntu Server, Windows endpoints, and VMware Workstation Pro.

This repository documents the complete deployment, troubleshooting, maintenance, and detection engineering activities performed while learning Security Operations, Threat Detection, and SIEM administration.

---

# Lab Objectives

The purpose of this lab is to gain practical experience with:

* SIEM deployment
* Endpoint monitoring
* Log collection
* Security event analysis
* Threat hunting
* Incident investigation
* Wazuh administration
* Linux administration
* Detection engineering

---

# Lab Environment

## Hypervisor

VMware Workstation Pro

## Systems

| System                    | Role                  |
| ------------------------- | --------------------- |
| Ubuntu Server 22.04.5 LTS | Wazuh Server          |
| Windows 10                | Monitored Endpoint    |
| Kali Linux                | Security Testing      |
| Additional VMs            | Future Detection Labs |

---

# Wazuh Architecture

```text
Windows Endpoint
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
Wazuh Indexer
        |
        |
        v
Wazuh Dashboard
```

---

# Repository Structure

```text
Wazuh/
│
├── README.md
│
├── Wazuh-Installation/
│   ├── wazuh-all-in-one-installation.md
│   ├── windows-agent-installation.md
│   └── wazuh-upgrade-4.7.5-to-4.14.5.md
│
├── Wazuh-Troubleshooting/
│   └── case-study-001-wazuh-dashboard-api-recovery.md
│
└── Wazuh-Detection-Labs/
```

---

# Installation Projects

## WAZUH-INSTALL-001

### Wazuh All-in-One Installation

Topics Covered:

* Ubuntu Server deployment
* Wazuh installation
* Service validation
* Dashboard access
* Network verification
* Snapshot strategy

Status:

```text
Completed
```

---

## WAZUH-INSTALL-002

### Windows Agent Installation and Enrollment

Topics Covered:

* Windows VM deployment
* VMware Tools
* Agent installation
* Agent enrollment
* Service verification
* Version compatibility troubleshooting

Status:

```text
Completed
```

---

## WAZUH-INSTALL-003

### Wazuh Upgrade (4.7.5 → 4.14.5)

Topics Covered:

* Package upgrades
* Service validation
* Snapshot strategy
* Configuration management
* Compatibility resolution

Status:

```text
Completed
```

---

# Troubleshooting Case Studies

## WAZUH-TROUBLESHOOTING-001

### Dashboard API Recovery After Upgrade

Problem:

```text
Application Not Found
```

Investigation Included:

* Service analysis
* Dashboard troubleshooting
* API troubleshooting
* Port validation
* Root cause analysis

Outcome:

```text
Resolved
```

---

# Skills Demonstrated

## Linux

* Package Management
* Service Management
* System Administration
* Log Analysis
* Troubleshooting

---

## Wazuh

* Installation
* Upgrade Management
* Agent Enrollment
* Dashboard Administration
* API Troubleshooting
* Configuration Management

---

## SOC Operations

* Security Monitoring
* Event Investigation
* Root Cause Analysis
* Technical Documentation
* Validation Testing

---

# Current Progress

| Project                   | Status    |
| ------------------------- | --------- |
| WAZUH-INSTALL-001         | Completed |
| WAZUH-INSTALL-002         | Completed |
| WAZUH-INSTALL-003         | Completed |
| WAZUH-TROUBLESHOOTING-001 | Completed |

---

# Upcoming Labs

## Detection Engineering

Planned Labs:

* Linux Authentication Monitoring
* Failed SSH Login Detection
* Brute Force Detection
* File Integrity Monitoring
* Threat Hunting
* MITRE ATT&CK Mapping
* Sysmon Integration
* Windows Event Monitoring

---

# Learning Outcomes

This lab provides practical experience in:

* Deploying a SIEM platform
* Managing endpoints
* Investigating failures
* Performing upgrades
* Troubleshooting production issues
* Preparing for SOC Analyst and Security Engineer roles

---

# Author

Aditya Raj

SOC Learning Portfolio

Focus Areas:

* Security Operations Center (SOC)
* SIEM Engineering
* Threat Detection
* Incident Response
* Digital Forensics
