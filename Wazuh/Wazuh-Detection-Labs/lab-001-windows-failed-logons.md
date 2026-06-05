# Lab 001 — Windows Failed Logons Detection & Investigation

## Objective

Generate failed Windows logon events and verify that Wazuh detects, processes, and alerts on authentication failures.

This lab demonstrates the complete detection lifecycle:

User Action → Windows Event Log → Wazuh Agent → Wazuh Rule → Security Alert → Analyst Investigation

---

## Lab Environment

| Component | Role | IP |
|------------|------------|------------|
| Ubuntu Server | Wazuh Manager + API + Dashboard | 192.168.21.135 |
| Windows 10 Pro | Monitored Endpoint | 192.168.21.136 |
| Kali Linux | Security Testing Workstation | Monitored by Wazuh |

---

## MITRE ATT&CK Mapping

| Technique | ID |
|------------|------------|
| Account Access Removal | T1531 |

Wazuh automatically mapped the alert to MITRE ATT&CK.

---

# Phase 1 — Generate Failed Authentication Events

## System

Windows 10 Endpoint

## Action

At the Windows login screen:

Entered an incorrect password multiple times for:

```text
employee1
```

Failed attempts generated:

```text
4
```

---

# Phase 2 — Validate Windows Event Logs

## Open

```text
Event Viewer
→ Windows Logs
→ Security
```

## Observed Event

```text
Event ID: 4625
```

Meaning:

```text
An account failed to log on
```

---



# Phase 3 — Verify Wazuh Detection

## Open

```text
Wazuh Dashboard
→ Threat Hunting
→ WIN10-ENDPOINT
```

Observed alerts:

```text
Logon Failure - Unknown user or bad password
```

Rule triggered:

```text
60122
```

Severity:

```text
Level 5
```

---



# Phase 4 — Investigate Alert Details

Opened alert details and reviewed fields.

---

## Rule Information

| Field | Value |
|---------|---------|
| Rule ID | 60122 |
| Level | 5 |
| Description | Logon Failure - Unknown user or bad password |

---

## Event Information

| Field | Value |
|---------|---------|
| Event ID | 4625 |
| Computer | DESKTOP-S046C93 |
| Username | employee1 |
| Source IP | 127.0.0.1 |
| Logon Type | 2 |
| Attempts | 4 |

---

# Understanding The Evidence

## Username

```text
employee1
```

This is the account Windows attempted to authenticate.

---

## Event ID

```text
4625
```

Windows Security Event:

```text
Failed Authentication
```

---

## Logon Type

```text
2
```

Meaning:

```text
Interactive Login
```

User entered credentials directly at the Windows login screen.

---

## Source IP

```text
127.0.0.1
```

Meaning:

```text
localhost
```

Authentication originated from the same host.

Not a remote system.

---

## Rule Fired Times

```text
4
```

This matched the four incorrect password attempts performed during testing.

---

# Detection Flow

```text
User enters incorrect password
        ↓
Windows creates Event ID 4625
        ↓
Wazuh Agent collects log
        ↓
Wazuh Rule 60122 matches
        ↓
Alert generated
        ↓
SOC Analyst investigates
```

---

# Analyst Investigation

## Questions Asked

### Who?

```text
employee1
```

---

### What Happened?

```text
Failed Windows authentication
```

---

### How Many Times?

```text
4
```

---

### From Where?

```text
127.0.0.1
(Localhost)
```

---

### Login Type?

```text
Interactive Login
(Logon Type 2)
```

---

### Malicious?

No evidence of malicious activity.

The activity was generated intentionally during lab testing.

---

# Analyst Notes

## Incident Title

```text
Multiple Failed Authentication Attempts
```

## Summary

Wazuh detected four failed authentication attempts against the local Windows endpoint.

## User

```text
employee1
```

## Source

```text
127.0.0.1
```

## Event ID

```text
4625
```

## Rule

```text
60122
```

## Severity

```text
5
```

## Impact

```text
Low
```

## Verdict

```text
Benign User Activity
(Lab Generated)
```

## Status

```text
Closed
```

---

# Validation Checklist

| Validation | Status |
|------------|------------|
| Windows Agent Installed | ✅ |
| Windows Logs Collected | ✅ |
| Event ID 4625 Generated | ✅ |
| Wazuh Alert Generated | ✅ |
| Rule 60122 Triggered | ✅ |
| Username Identified | ✅ |
| Source Identified | ✅ |
| Logon Type Identified | ✅ |
| Alert Investigated | ✅ |
| Detection Validated | ✅ |

---

# Skills Demonstrated

- Windows Security Logging
- Event Viewer Analysis
- Authentication Monitoring
- Wazuh Threat Hunting
- Alert Investigation
- SOC Triage Process
- MITRE ATT&CK Mapping
- Security Event Analysis

---

# Lessons Learned

1. Failed logons generate Event ID 4625.
2. Wazuh correctly parses Windows Security logs.
3. Rule 60122 detects authentication failures.
4. Analysts must investigate context, not just alerts.
5. Logon Type and Source IP help determine attack origin.
6. Multiple failed logons can indicate brute-force activity.
7. Proper triage reduces false positives.

---

## Result

Successfully generated, detected, and investigated Windows failed authentication events using Wazuh SIEM.
