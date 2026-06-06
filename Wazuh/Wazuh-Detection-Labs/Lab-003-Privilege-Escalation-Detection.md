# Lab 003 — Privilege Escalation Detection

## Objective

Detect a local privilege escalation event where a standard user account is added to the local Administrators group and investigate the resulting Windows and Wazuh alerts.

---

# Lab Environment

## Systems

| System        | IP Address     | Role                          |
| ------------- | -------------- | ----------------------------- |
| Ubuntu Server | 192.168.21.135 | Wazuh Manager                 |
| Windows 10    | 192.168.21.136 | Monitored Endpoint            |
| Kali Linux    | 192.168.21.130 | Attacker/Investigation System |

---

# MITRE ATT&CK Mapping

| Technique ID | Name                 |
| ------------ | -------------------- |
| T1098        | Account Manipulation |

---

# Scenario

After creating a new local user account, an administrator adds that account to the local Administrators group.

Such activity may indicate:

* Privilege escalation
* Persistence
* Unauthorized account manipulation
* Insider threat activity

---

# Step 1 — Verify Current Administrative Users

## Run On: Windows

```cmd
net localgroup administrators
```

Output:

```text
Administrator
attackerlab
employee1
```

---

# Step 2 — Add User To Administrators Group

## Run On: Windows (Administrator PowerShell/CMD)

```cmd
net localgroup administrators attackerlab /add
```

Purpose:

* Add user to local Administrators group
* Generate Windows Security Event 4732

---

# Understanding The Command

```cmd
net localgroup administrators attackerlab /add
```

| Component      | Description                        |
| -------------- | ---------------------------------- |
| net            | Windows account management utility |
| localgroup     | Manage local groups                |
| administrators | Target group                       |
| attackerlab    | User being modified                |
| /add           | Action performed                   |

Linux equivalent:

```bash
sudo usermod -aG sudo attackerlab
```

---

# Step 3 — Validate Membership

## Run On: Windows

```cmd
net user attackerlab
```

Relevant Output:

```text
Local Group Memberships
*Administrators
*Users
```

Result:

User successfully received administrative privileges.

---

# Step 4 — Investigate Windows Event Logs

## Run On: Windows

Open:

```text
Event Viewer
→ Windows Logs
→ Security
```

Filter:

```text
Event ID: 4732
```

Observed Event:

```text
Event ID: 4732
A member was added to a security-enabled local group
```

---

# Event Analysis

## Subject

```text
employee1
```

User performing the action.

---

## Modified Account

```text
attackerlab
```

User receiving elevated privileges.

---

## Group

```text
Administrators
```

Target security group.

---

# Event Evidence

| Field        | Value                     |
| ------------ | ------------------------- |
| Event ID     | 4732                      |
| Subject User | employee1                 |
| Added User   | attackerlab               |
| Group Name   | Administrators            |
| Category     | Security Group Management |

---

# Step 5 — Investigate In Wazuh

Open:

```text
Threat Hunting
```

Search:

```text
4732
```

or

```text
Administrators
```

---

# Wazuh Alert Details

| Field        | Value                        |
| ------------ | ---------------------------- |
| Rule ID      | 60154                        |
| Rule Level   | 12                           |
| Description  | Administrators Group Changed |
| Subject User | employee1                    |
| Target Group | Administrators               |

---

# Detection Flow

```text
employee1
        ↓
Added attackerlab to Administrators
        ↓
Windows Event 4732 Generated
        ↓
Wazuh Agent Collected Event
        ↓
Rule 60154 Matched
        ↓
Alert Generated
        ↓
SOC Investigation
```

---

# SOC Analyst Investigation

Questions to ask:

1. Was the account addition authorized?
2. Who performed the change?
3. Was a change ticket approved?
4. Is the new administrator account legitimate?
5. Are there related persistence activities?

---

# Lessons Learned

* Windows logs administrator group changes using Event ID 4732.
* Wazuh detects local administrator group modifications.
* Privilege escalation events carry higher security risk than simple authentication failures.
* User account monitoring is critical for detecting persistence and unauthorized privilege assignments.

---

# Skills Developed

## Windows

* Event Viewer Analysis
* User Management
* Group Management

## Wazuh

* Threat Hunting
* Alert Investigation
* Rule Analysis

## Detection Engineering

* Privilege Escalation Detection
* Account Manipulation Monitoring
* MITRE ATT&CK Mapping

---

# Conclusion

This lab demonstrated how administrator group modifications generate Windows Security Event 4732 and trigger Wazuh Rule 60154. The activity was successfully detected, investigated, and mapped to MITRE ATT&CK Technique T1098 (Account Manipulation), providing a realistic privilege escalation detection use case for SOC and Detection Engineering workflows.
