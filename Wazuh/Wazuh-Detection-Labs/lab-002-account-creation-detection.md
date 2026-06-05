# Lab 002 — Windows Account Creation Detection (Event ID 4720)

## Objective

The objective of this lab was to simulate the creation of a new local Windows account, investigate the resulting Windows Security Event, verify SIEM ingestion in Wazuh, and understand how account creation activity can be detected as a potential persistence mechanism.

This lab demonstrates an end-to-end detection workflow from activity generation to SIEM investigation.

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
| Windows Endpoint IP | 192.168.21.136        |
| Kali Linux IP       | 192.168.21.130        |

---

# Attack Scenario

An administrator creates a new local account on a Windows workstation.

In real-world attacks, adversaries commonly create local accounts to:

* Establish persistence
* Maintain long-term access
* Create backup accounts
* Prepare for privilege escalation
* Blend into legitimate administrative activity

MITRE ATT&CK maps this activity to:

| Category  | Value          |
| --------- | -------------- |
| Tactic    | Persistence    |
| Technique | T1136          |
| Name      | Create Account |

---

# Step 1 — Create Local User Account

## Run on Windows

```cmd
net user attackerlab P@ssw0rd123! /add
```

### Command Breakdown

#### net user

Used to manage local Windows accounts.

#### attackerlab

Username to create.

#### P@ssw0rd123!

Password assigned to the account.

#### /add

Creates the account.

---

# Step 2 — Verify Account Creation

## Run on Windows

```cmd
net user attackerlab
```

### Output Observed

```text
User name: attackerlab
Account active: Yes
Local Group Memberships: Users
Last logon: Never
```

### Analysis

The account was successfully created and exists as a standard local user.

At this stage the account has not been granted administrative privileges.

---

# Step 3 — Investigate Windows Security Logs

## Run on Windows

Open:

```text
Event Viewer
→ Windows Logs
→ Security
```

Filter for:

```text
Event ID 4720
```

---

# Windows Event Collected

### Event ID

```text
4720
```

### Description

```text
A user account was created.
```

---

# Evidence Collected

## Creator Account

```text
employee1
```

Windows recorded that the account creation action was performed by:

```text
employee1
```

---

## Created Account

```text
attackerlab
```

Windows recorded that the following account was created:

```text
attackerlab
```

---

## Event Timestamp

```text
05-06-2026 04:43:25
```

---

# Windows Investigation Findings

Windows Security Logging provides:

* Who created the account
* Which account was created
* When the account was created
* Which host performed the action

This information is critical during incident response investigations.

---

# Step 4 — Verify SIEM Detection

## Open Wazuh Dashboard

Navigate to:

```text
Threat Hunting
```

Search for:

```text
data.win.system.eventID:4720
```

---

# Wazuh Detection Results

## Rule ID

```text
60109
```

---

## Rule Level

```text
8
```

---

## Subject User

```text
data.win.eventdata.subjectUserName

employee1
```

---

## Target User

```text
data.win.eventdata.targetUserName

attackerlab
```

---

# Detection Logic

The detection can be simplified as:

```text
IF
    Windows Event ID = 4720

THEN
    Generate Alert

BECAUSE
    A local user account has been created
```

---

# Detection Flow

```text
Administrator creates account

        ↓

net user attackerlab /add

        ↓

Windows generates Event ID 4720

        ↓

Wazuh Agent collects Security Event

        ↓

Event sent to Wazuh Manager

        ↓

Rule 60109 matches

        ↓

Alert generated

        ↓

SOC Analyst investigates
```

---

# Security Analysis

Creating user accounts is a common persistence technique.

Attackers rarely create accounts with obvious names.

Examples:

```cmd
net user backupsvc Password123! /add
```

```cmd
net user helpdesk Password123! /add
```

```cmd
net user supportsvc Password123! /add
```

Such names often blend into enterprise environments.

---

# SOC Analyst Investigation Questions

If this alert appeared in production, an analyst should ask:

1. Was the account creation authorized?
2. Was the creator account expected to perform administrative tasks?
3. Was the account created during a maintenance window?
4. Was the account subsequently added to Administrators?
5. Did the account log in after creation?
6. Was the account later used remotely?

---

# MITRE ATT&CK Mapping

## Tactic

```text
Persistence
```

## Technique

```text
T1136
```

## Name

```text
Create Account
```

## Reason

Adversaries may create local accounts to maintain access after initial compromise.

---

# Evidence Summary

| Artifact        | Value       |
| --------------- | ----------- |
| Event ID        | 4720        |
| Rule ID         | 60109       |
| Rule Level      | 8           |
| Creator Account | employee1   |
| Created Account | attackerlab |
| Technique       | T1136       |
| Tactic          | Persistence |

---

# Lessons Learned

* Windows generates Event ID 4720 when a local account is created.
* Wazuh successfully collects and parses account creation events.
* Rule 60109 detects local user account creation activity.
* Event data contains both creator and target account information.
* Account creation is a common persistence technique.
* Correlating Windows Events and SIEM alerts improves investigation accuracy.
* Understanding raw Windows events is essential for detection engineering.

---

# Screenshots

## Screenshot 1

Event Viewer showing Event ID 4720.

## Screenshot 2

Event Viewer showing creator account and target account.

## Screenshot 3

Wazuh Threat Hunting search for Event ID 4720.

## Screenshot 4

Wazuh alert details showing:

* Rule ID 60109
* Rule Level 8
* subjectUserName = employee1
* targetUserName = attackerlab

---

# Conclusion

This lab demonstrated the complete lifecycle of detecting Windows account creation activity. A new local account was created, recorded by Windows Security Logs, collected by the Wazuh Agent, processed by the Wazuh Manager, and surfaced as Rule 60109 in the SIEM. This workflow reflects a real-world persistence detection use case commonly investigated by SOC Analysts, Detection Engineers, and DFIR practitioners.
