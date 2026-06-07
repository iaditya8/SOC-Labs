# Lab 005 — Process Creation Monitoring (Event ID 4688)

## Objective

The objective of this lab was to enable Windows Process Creation Auditing, generate process execution activity, investigate the resulting Windows Security Event, verify SIEM ingestion in Wazuh, and understand how process creation events can be used for threat hunting, detection engineering, and incident response.

This lab demonstrates an end-to-end detection workflow from process execution to SIEM investigation.

---

# Lab Environment

| Component           | Details               |
| ------------------- | --------------------- |
| Wazuh Manager       | Ubuntu Server 22.04   |
| Wazuh Version       | 4.14.5                |
| Windows Endpoint    | Windows 10 Pro        |
| Windows Agent ID    | 002                   |
| Kali Linux          | Kali GNU/Linux 2025.4 |
| Kali Agent ID       | 003                   |
| Wazuh Manager IP    | 192.168.21.135        |
| Windows Endpoint IP | 192.168.21.143        |
| Kali Linux IP       | 192.168.21.130        |

---

# Attack Scenario

An attacker or administrator executes commands on a Windows system.

In real-world environments, adversaries execute processes to:

* Enumerate system information
* Discover users and groups
* Perform reconnaissance
* Execute malware
* Launch PowerShell payloads
* Maintain persistence

MITRE ATT&CK maps this activity to multiple techniques depending on the executed process.

Process creation monitoring serves as a foundational telemetry source for detecting attacker behavior.

---

# Step 1 — Verify Process Creation Auditing

## Run on Windows

```cmd
auditpol /get /subcategory:"Process Creation"
```

### Command Breakdown

#### auditpol

Windows auditing configuration utility.

#### /get

Displays the current audit configuration.

#### Process Creation

Controls logging for Event ID 4688.

### Initial Output

```text
Process Creation    No Auditing
```

### Analysis

Windows was not configured to generate Process Creation events.

Without enabling this audit policy, Event ID 4688 would not be generated.

---

# Step 2 — Enable Process Creation Auditing

## Run on Windows

```cmd
auditpol /set /subcategory:"Process Creation" /success:enable
```

### Command Breakdown

#### /set

Modifies audit policy settings.

#### /success:enable

Logs successful process creation events.

---

# Step 3 — Verify Configuration

## Run on Windows

```cmd
auditpol /get /subcategory:"Process Creation"
```

### Output Observed

```text
Process Creation    Success
```

### Analysis

Windows was successfully configured to generate Event ID 4688 whenever a process starts.

---

# Step 4 — Generate Process Activity

## Run on Windows

```cmd
whoami
```

```cmd
hostname
```

```cmd
ipconfig
```

```cmd
net user
```

```cmd
tasklist
```

### Purpose

These commands simulate basic system enumeration activity commonly performed by administrators and attackers.

---

# Step 5 — Investigate Windows Security Logs

## Open Event Viewer

```text
Event Viewer
→ Windows Logs
→ Security
```

Filter for:

```text
Event ID 4688
```

---

# Windows Event Collected

### Event ID

```text
4688
```

### Description

```text
A new process has been created.
```

---

# Evidence Collected

## New Process Name

```text
C:\Windows\System32\tasklist.exe
```

---

## Additional Process Observed

```text
C:\Windows\System32\net1.exe
```

---

## Creator Account

```text
employee1
```

---

## Elevation Level

```text
High Mandatory Level
```

---

# Windows Investigation Findings

Windows Security Logging provides:

* Process Name
* Parent Process
* User Context
* Integrity Level
* Process Creation Time

This information is critical for threat hunting and incident response investigations.

---

# Process Tree Analysis

### Tasklist Execution

```text
employee1
      ↓
tasklist.exe
```

---

### Net Command Execution

```text
employee1
      ↓
net.exe
      ↓
net1.exe
```

### Analysis

Many Windows commands launch helper processes internally.

Analysts must understand process relationships when performing investigations.

---

# Step 6 — Verify SIEM Detection

## Open Wazuh Dashboard

Navigate to:

```text
Threat Hunting
```

Search for:

```text
data.win.system.eventID:4688
```

---

# Wazuh Detection Results

## Rule ID

```text
67027
```

---

## Rule Level

```text
3
```

---

## Rule Description

```text
A process was created.
```

---

## Process Name

```text
C:\Windows\System32\taskhostw.exe
```

---

## Parent Process

```text
C:\Windows\System32\svchost.exe
```

---

## Target User

```text
employee1
```

---

# Detection Logic

The detection can be simplified as:

```text
IF
    Windows Event ID = 4688

THEN
    Generate Alert

BECAUSE
    A new process was created
```

---

# Detection Flow

```text
User executes command

        ↓

Windows creates process

        ↓

Event ID 4688 generated

        ↓

Wazuh Agent collects event

        ↓

Event sent to Wazuh Manager

        ↓

Rule 67027 matches

        ↓

Alert generated

        ↓

SOC Analyst investigates
```

---

# Security Analysis

Process Creation Monitoring is one of the most valuable Windows telemetry sources.

It enables detection of:

* PowerShell execution
* Command Prompt usage
* Living-off-the-Land binaries
* Malware execution
* Privilege escalation activity
* Lateral movement preparation

Examples:

```cmd
powershell.exe
```

```cmd
cmd.exe
```

```cmd
certutil.exe
```

```cmd
wmic.exe
```

```cmd
rundll32.exe
```

```cmd
mshta.exe
```

---

# SOC Analyst Investigation Questions

If this alert appeared in production, an analyst should ask:

1. Which process was executed?
2. Which user launched the process?
3. Was the process expected on the system?
4. What was the parent process?
5. Was the process executed with elevated privileges?
6. Does the process indicate attacker reconnaissance?
7. Is the activity associated with known malicious behavior?

---

# MITRE ATT&CK Mapping

## Tactic

```text
Execution
```

---

## Technique

```text
T1059
```

---

## Name

```text
Command and Scripting Interpreter
```

---

## Reason

Adversaries frequently execute commands and scripts during reconnaissance, execution, and post-exploitation phases.

---

# Evidence Summary

| Artifact           | Value        |
| ------------------ | ------------ |
| Event ID           | 4688         |
| Rule ID            | 67027        |
| Rule Level         | 3            |
| User               | employee1    |
| Process            | tasklist.exe |
| Additional Process | net1.exe     |
| Technique          | T1059        |
| Tactic             | Execution    |

---

# Lessons Learned

* Windows generates Event ID 4688 when a process starts.
* Process Creation Auditing must be enabled before telemetry is collected.
* Wazuh successfully collects and parses Event ID 4688.
* Rule 67027 detects process creation activity.
* Parent-child process relationships are critical during investigations.
* Process creation telemetry forms the foundation of threat hunting and detection engineering.
* Understanding process execution is essential for SOC, DFIR, and Detection Engineering roles.

---



# Conclusion

This lab demonstrated the complete lifecycle of Process Creation Monitoring using Windows Event ID 4688. Process execution activity was generated, recorded by Windows Security Logs, collected by the Wazuh Agent, processed by the Wazuh Manager, and surfaced as Rule 67027 in the SIEM. Process creation telemetry provides critical visibility into user actions, system activity, and potential attacker behavior, making it one of the most important data sources for SOC Analysts, Threat Hunters, Detection Engineers, and DFIR practitioners.
