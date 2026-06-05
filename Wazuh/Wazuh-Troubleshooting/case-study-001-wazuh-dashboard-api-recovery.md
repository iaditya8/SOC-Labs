# Case Study 001 - Wazuh Dashboard API Recovery After Upgrade

## Project Information

| Field            | Value                                      |
| ---------------- | ------------------------------------------ |
| Project ID       | WAZUH-TROUBLESHOOTING-001                  |
| Title            | Wazuh Dashboard API Recovery After Upgrade |
| Category         | Wazuh Troubleshooting                      |
| Difficulty       | Intermediate                               |
| Status           | Completed                                  |
| Environment      | VMware Workstation Pro                     |
| Wazuh Version    | 4.14.5                                     |
| Operating System | Ubuntu Server 22.04.5 LTS                  |

---

# Executive Summary

During the setup and maintenance of a Wazuh SIEM lab environment, the Wazuh Dashboard became inaccessible after an upgrade process.

Users were able to access the login page and authenticate successfully. However, after login, the dashboard displayed:

```text
Application Not Found
```

The issue prevented access to all Wazuh functionality, including:

* Threat Hunting
* Vulnerability Detection
* Configuration Assessment
* File Integrity Monitoring
* Agent Management

A structured troubleshooting methodology was followed to identify the root cause and restore functionality.

The investigation revealed that the Wazuh API service was not listening on port 55000, preventing communication between the dashboard and the Wazuh Manager.

The issue was successfully resolved and validated.

---

# Environment

## Hypervisor

VMware Workstation Pro

## Host Machine

Windows 10

## Virtual Machines Used

### Wazuh Server

| Property         | Value                            |
| ---------------- | -------------------------------- |
| Hostname         | wazuh                            |
| Operating System | Ubuntu Server 22.04.5 LTS        |
| IP Address       | 192.168.21.135                   |
| Deployment Type  | All-in-One                       |
| Components       | Dashboard, Manager, Indexer, API |
| Version          | 4.14.5                           |

### Administration Workstation

| Property         | Value                         |
| ---------------- | ----------------------------- |
| Operating System | Windows 10                    |
| Purpose          | Dashboard Access & Validation |

---

# Problem Statement

After logging into Wazuh Dashboard:

```text
https://192.168.21.135
```

The following error appeared:

```text
Application Not Found
```

The dashboard interface failed to load even though authentication was successful.

---

# Initial Hypotheses

The following possibilities were considered:

1. Missing Wazuh dashboard plugin
2. Broken dashboard route
3. Dashboard configuration corruption
4. API connectivity failure
5. Version mismatch after upgrade
6. Indexer communication failure

---

# Investigation Timeline

## Step 1 - Verify Dashboard Configuration File

### Command

```bash
sudo cat /etc/wazuh-dashboard/opensearch_dashboard.yml
```

### Result

```text
No such file or directory
```

### Analysis

The filename was incorrect.

---

## Step 2 - Discover Existing Dashboard Configuration Files

### Command

```bash
sudo ls -l /etc/wazuh-dashboard/
```

### Command

```bash
sudo find /etc/wazuh-dashboard -type f
```

### Finding

Located the correct configuration file:

```text
/etc/wazuh-dashboard/opensearch_dashboards.yml
```

### Conclusion

Dashboard configuration existed.

---

## Step 3 - Verify Dashboard Plugin Installation

### Command

```bash
cat /usr/share/wazuh-dashboard/plugins/wazuh/package.json | head -20
```

### Output

```json
{
  "name": "wazuh",
  "version": "4.14.5"
}
```

### Analysis

Plugin installation was successful.

### Conclusion

Plugin corruption was ruled out.

---

## Step 4 - Verify Dashboard Routing

### Command

```bash
sudo cat /etc/wazuh-dashboard/opensearch_dashboards.yml
```

### Finding

```yaml
uiSettings.overrides.defaultRoute: /app/wazuh
```

### Analysis

Dashboard route was configured.

---

## Step 5 - Direct Route Testing

### URL

```text
https://192.168.21.135/app/wazuh
```

### Result

```text
Application Not Found
```

### Analysis

Routing was not the primary issue.

---

## Step 6 - Verify API Connectivity

Dashboard displayed:

```text
Host: https://localhost
Port: 55000
Status: Offline
```

### Analysis

Dashboard could not communicate with the Wazuh API.

---

## Step 7 - Verify API Listening Port

### Command

```bash
sudo ss -tulpn | grep 55000
```

### Result

No output returned.

### Expected

```text
LISTEN
```

### Analysis

Nothing was listening on port 55000.

### Critical Discovery

This became the primary investigation path.

---

## Step 8 - Verify Wazuh Manager Health

### Command

```bash
sudo systemctl status wazuh-manager
```

### Result

```text
active (running)
```

### Analysis

Manager service healthy.

### Conclusion

Problem isolated to API layer.

---

## Step 9 - Review Manager Logs

### Command

```bash
sudo journalctl -u wazuh-manager -n 100 --no-pager
```

### Findings

Manager started successfully.

Observed warnings:

```text
provider
run_on_start
min_full_scan_interval
```

### Analysis

Warnings did not impact startup.

Manager functionality remained intact.

---

## Step 10 - Verify API Configuration

### Command

```bash
sudo ls -l /var/ossec/api/configuration/
```

### Output

```text
api.yaml
security/
ssl/
```

### Analysis

API configuration files present.

---

## Step 11 - Inspect API Configuration

### Command

```bash
sudo cat /var/ossec/api/configuration/api.yaml
```

### Analysis

Configuration existed and appeared healthy.

No corruption detected.

---

## Step 12 - Verify API Process

### Command

```bash
sudo ps aux | grep wazuh-apid
```

### Output

```text
grep --color=auto wazuh-apid
```

### Analysis

API process not running.

### Conclusion

Confirmed API service failure.

---

## Step 13 - Locate API Binary

### Command

```bash
sudo find /var/ossec -name "*apid*"
```

### Output

```text
/var/ossec/bin/wazuh-apid
```

### Analysis

API executable exists.

Binary corruption ruled out.

---

## Step 14 - Manual API Startup

### Command

```bash
sudo /var/ossec/bin/wazuh-apid -f
```

### Purpose

Run API in foreground mode for troubleshooting.

### Output

```text
INFO: Starting API in foreground
INFO: Checking RBAC database integrity
INFO: Listening on [::]:55000
INFO: Getting installation UID
INFO: Getting updates information
```

### Analysis

API started successfully.

No:

* SSL errors
* Permission issues
* Database corruption
* Configuration problems

were observed.

---

# Root Cause Analysis

The Wazuh Dashboard depends on the Wazuh API.

Normal communication path:

```text
Dashboard
    |
    v
API (55000)
    |
    v
Manager
    |
    v
Indexer
```

The API service was not running.

As a result:

```text
Dashboard
    |
    X
API Offline
    |
    X
Port 55000 Closed
    |
    X
Application Not Found
```

### Root Cause

The Wazuh API process was not running after the upgrade process.

Because the API was unavailable:

* Dashboard could authenticate users
* Dashboard could not communicate with the manager
* Application routes failed
* Dashboard functionality became unavailable

---

# Recovery Procedure

## Start API

```bash
sudo /var/ossec/bin/wazuh-apid -f
```

---

## Verify Port

```bash
sudo ss -tulpn | grep 55000
```

### Result

```text
LISTEN 0 2048 0.0.0.0:55000
LISTEN 0 2048 [::]:55000
```

---

## Refresh Dashboard

Dashboard loaded successfully.

Modules available:

* Threat Hunting
* Vulnerability Detection
* MITRE ATT&CK
* File Integrity Monitoring
* Configuration Assessment
* PCI DSS
* Agent Management

---

# Persistence Validation

## Restart Manager

### Command

```bash
sudo systemctl restart wazuh-manager
```

### Purpose

Validate persistence.

---

## Verify Port Again

### Command

```bash
sudo ss -tulpn | grep 55000
```

### Result

```text
LISTEN 0 2048 0.0.0.0:55000
LISTEN 0 2048 [::]:55000
```

### Analysis

API restarted successfully.

### Conclusion

Fix survived service restart.

---

# Commands Reference

## Dashboard Configuration

```bash
sudo find /etc/wazuh-dashboard -type f
```

Locate dashboard configuration files.

---

```bash
sudo cat /etc/wazuh-dashboard/opensearch_dashboards.yml
```

Inspect dashboard configuration.

---

## Plugin Validation

```bash
cat /usr/share/wazuh-dashboard/plugins/wazuh/package.json
```

Verify plugin installation.

---

## Manager Verification

```bash
sudo systemctl status wazuh-manager
```

Verify manager health.

---

```bash
sudo journalctl -u wazuh-manager -n 100 --no-pager
```

Review startup logs.

---

## API Validation

```bash
sudo ss -tulpn | grep 55000
```

Verify API listening port.

---

```bash
sudo ps aux | grep wazuh-apid
```

Verify API process.

---

```bash
sudo find /var/ossec -name "*apid*"
```

Locate API executable.

---

```bash
sudo /var/ossec/bin/wazuh-apid -f
```

Run API in foreground mode.

---

# Skills Demonstrated

* Linux Administration
* Wazuh Administration
* SIEM Troubleshooting
* Service Analysis
* API Troubleshooting
* Log Analysis
* Root Cause Analysis
* Incident Documentation
* Validation Testing
* Security Infrastructure Maintenance

---

# Lessons Learned

1. Dashboard failures are not always dashboard problems.
2. Port 55000 is the fastest API health indicator.
3. Validate services before modifying configurations.
4. Foreground execution is extremely useful for troubleshooting.
5. Verify fixes survive restarts.
6. Troubleshoot systematically:

   * Dashboard
   * Route
   * Plugin
   * API
   * Service
   * Port
7. Configuration files should never be modified before collecting evidence.

---

# SOC Relevance

This troubleshooting exercise demonstrates practical skills required for:

* SOC Analyst
* SIEM Engineer
* Security Engineer
* Detection Engineer
* Security Operations Engineer

The exercise involved:

* Infrastructure troubleshooting
* Service validation
* API troubleshooting
* Linux administration
* Evidence-based investigation
* Root cause analysis

---

# Final Status

| Component              | Status     |
| ---------------------- | ---------- |
| Dashboard              | Healthy    |
| Plugin                 | Healthy    |
| API                    | Healthy    |
| Manager                | Healthy    |
| Indexer                | Healthy    |
| Port 55000             | Listening  |
| Dashboard Access       | Working    |
| Persistence Validation | Successful |

---

# Project Outcome

```text
Project ID : WAZUH-TROUBLESHOOTING-001
Title      : Wazuh Dashboard API Recovery After Upgrade
Status     : COMPLETED
Result     : SUCCESS
```
