# Wazuh Upgrade Guide

## Upgrading Wazuh from 4.7.5 to 4.14.5

---

# Project Information

| Field            | Value                              |
| ---------------- | ---------------------------------- |
| Project ID       | WAZUH-INSTALL-003                  |
| Title            | Wazuh Upgrade from 4.7.5 to 4.14.5 |
| Status           | Completed                          |
| Difficulty       | Intermediate                       |
| Environment      | VMware Workstation Pro             |
| Operating System | Ubuntu Server 22.04.5 LTS          |
| Upgrade Type     | In-Place Upgrade                   |

---

# Objective

Upgrade an existing Wazuh deployment from version 4.7.5 to 4.14.5 in order to:

* Support newer Wazuh agents
* Resolve agent enrollment incompatibilities
* Gain access to newer features
* Improve platform stability

---

# Background

While attempting to enroll a Windows endpoint, the following error was observed:

```text
ERROR: Incompatible version for new agent
```

Investigation revealed:

| Component     | Version |
| ------------- | ------- |
| Wazuh Manager | 4.7.5   |
| Windows Agent | 4.13.1  |

Because the agent version was newer than the manager version, enrollment was rejected.

---

# Initial Environment

## Wazuh Components

| Component       | Version |
| --------------- | ------- |
| Wazuh Dashboard | 4.7.5   |
| Wazuh Manager   | 4.7.5   |
| Wazuh Indexer   | 4.7.5   |

---

# Step 1 - Verify Current Versions

Command:

```bash
dpkg -l | grep wazuh
```

Purpose:

Display installed Wazuh packages.

Output:

```text
wazuh-dashboard 4.7.5
wazuh-indexer 4.7.5
wazuh-manager 4.7.5
```

---



# Step 2 - Verify Available Versions

Command:

```bash
apt-cache policy wazuh-manager
```

Purpose:

Display available package versions.

Finding:

```text
Candidate: 4.14.5
```

Analysis:

Upgrade path available through configured repository.

---

# Step 3 - Create VMware Snapshot

Before modifying production services:

```text
VM
└── Snapshot
    └── Take Snapshot
```

Snapshot Name:

```text
Before-Wazuh-Upgrade
```

Purpose:

Provide rollback capability if upgrade fails.

---



# Upgrade Strategy

Upgrade order:

```text
Indexer
   ↓
Manager
   ↓
Dashboard
```

Reason:

Dependencies should be upgraded from backend to frontend.

---

# Step 4 - Upgrade Wazuh Indexer

Command:

```bash
sudo apt update
```

---

Command:

```bash
sudo apt install wazuh-indexer=4.14.5-1
```

Purpose:

Upgrade Indexer component.

---

# Configuration Prompt

During installation:

```text
/etc/init.d/wazuh-indexer
```

Prompt:

```text
Install maintainer's version?
```

Selection:

```text
Y
```

Reason:

Upgrade init scripts to the latest version.

---

# Validation

Command:

```bash
sudo systemctl status wazuh-indexer
```

Expected:

```text
active (running)
```

---



# Step 5 - Upgrade Wazuh Manager

Command:

```bash
sudo apt install wazuh-manager=4.14.5-1
```

Purpose:

Upgrade manager component.

---

# Configuration Decisions

For:

```text
/etc/init.d/wazuh-manager
```

Selection:

```text
Y
```

Reason:

Use updated service startup script.

---

# Validation

Command:

```bash
sudo systemctl status wazuh-manager
```

Expected:

```text
active (running)
```

---



# Step 6 - Upgrade Dashboard

Command:

```bash
sudo apt install wazuh-dashboard=4.14.5-1
```

Purpose:

Upgrade dashboard component.

---

# Configuration Prompt

File:

```text
/etc/wazuh-dashboard/opensearch_dashboards.yml
```

Selection:

```text
N
```

Reason:

Keep existing working configuration.

Avoid overwriting custom settings.

---

# Validation

Command:

```bash
sudo systemctl status wazuh-dashboard
```

Expected:

```text
active (running)
```

---



# Step 7 - Verify Upgrade

Command:

```bash
dpkg -l | grep wazuh
```

Expected:

```text
wazuh-dashboard 4.14.5
wazuh-indexer 4.14.5
wazuh-manager 4.14.5
```

---



# Step 8 - Reboot System

Command:

```bash
sudo reboot
```

Purpose:

Ensure services start cleanly.

---

# Post-Upgrade Validation

## Verify Manager

```bash
sudo systemctl status wazuh-manager
```

---

## Verify Dashboard

```bash
sudo systemctl status wazuh-dashboard
```

---

## Verify Indexer

```bash
sudo systemctl status wazuh-indexer
```

---

# Disk Cleanup

After upgrade:

Clean downloaded package cache.

Command:

```bash
sudo apt clean
```

Purpose:

Remove downloaded package files.

---

Remove unused packages:

```bash
sudo apt autoremove -y
```

Purpose:

Remove obsolete dependencies.

---

Check journal size:

```bash
journalctl --disk-usage
```

---

Check Wazuh log size:

```bash
sudo du -sh /var/ossec/logs
```

---

# Snapshot Management

Review existing snapshots.

Remove unnecessary snapshots after successful validation.

Recommended Snapshot:

```text
Wazuh-Working-4.14.5
```

Purpose:

Stable rollback point.

---

# Issue Encountered After Upgrade

Dashboard became inaccessible.

Observed:

```text
Application Not Found
```

This issue was documented separately as:

```text
WAZUH-TROUBLESHOOTING-001
case-study-001-wazuh-dashboard-api-recovery.md
```

---



# Root Cause of Original Enrollment Problem

Before Upgrade:

| Component | Version |
| --------- | ------- |
| Manager   | 4.7.5   |
| Agent     | 4.13.1  |

Result:

```text
ERROR: Incompatible version for new agent
```

---

After Upgrade:

| Component | Version |
| --------- | ------- |
| Manager   | 4.14.5  |
| Agent     | 4.13.1  |

Result:

```text
Enrollment Successful
```

---

# Commands Reference

## Check Installed Versions

```bash
dpkg -l | grep wazuh
```

---

## Check Available Versions

```bash
apt-cache policy wazuh-manager
```

---

## Upgrade Indexer

```bash
sudo apt install wazuh-indexer=4.14.5-1
```

---

## Upgrade Manager

```bash
sudo apt install wazuh-manager=4.14.5-1
```

---

## Upgrade Dashboard

```bash
sudo apt install wazuh-dashboard=4.14.5-1
```

---

## Verify Services

```bash
sudo systemctl status wazuh-manager
```

```bash
sudo systemctl status wazuh-dashboard
```

```bash
sudo systemctl status wazuh-indexer
```

---

## Clean Package Cache

```bash
sudo apt clean
```

---

## Remove Unused Packages

```bash
sudo apt autoremove -y
```

---

# Lessons Learned

1. Agent versions must be compatible with manager versions.
2. Always create snapshots before upgrades.
3. Upgrade backend services before frontend services.
4. Preserve custom configuration files unless replacement is required.
5. Validate all services after upgrade.
6. Keep a rollback plan before making major changes.

---

# Skills Demonstrated

* Linux Administration
* Package Management
* Service Management
* Wazuh Administration
* Upgrade Planning
* Configuration Management
* Change Control
* Validation Testing

---

# Validation Checklist

| Task                           | Status    |
| ------------------------------ | --------- |
| Current Version Verified       | Completed |
| Repository Verified            | Completed |
| Snapshot Created               | Completed |
| Indexer Upgraded               | Completed |
| Manager Upgraded               | Completed |
| Dashboard Upgraded             | Completed |
| Services Validated             | Completed |
| Reboot Tested                  | Completed |
| Version Compatibility Resolved | Completed |

---

# Project Outcome

```text
Project ID : WAZUH-INSTALL-003
Title      : Wazuh Upgrade from 4.7.5 to 4.14.5
Status     : COMPLETED
Result     : SUCCESS
```
