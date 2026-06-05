# Kali Linux Agent Enrollment into Wazuh

## Overview

This document describes the installation, enrollment, and verification of a Kali Linux endpoint into the Wazuh SIEM environment.

The objective was to onboard Kali Linux as a monitored endpoint so that security events, system activity, configuration changes, and audit data could be collected and analyzed through the Wazuh Manager.

---

## Lab Architecture

```text
Ubuntu Server 22.04.5 LTS
IP: 192.168.21.135
Role:
- Wazuh Manager
- Wazuh API
- Wazuh Dashboard

        │
        │
        ├── Windows 10 Endpoint
        │      Agent ID: 002
        │
        └── Kali Linux 2025.4
               Agent ID: 003
```

---

## Environment Details

### Wazuh Manager

| Item             | Value                     |
| ---------------- | ------------------------- |
| Hostname         | wazuh                     |
| Operating System | Ubuntu Server 22.04.5 LTS |
| IP Address       | 192.168.21.135            |
| Wazuh Version    | 4.14.5                    |

### Kali Endpoint

| Item             | Value                  |
| ---------------- | ---------------------- |
| Hostname         | kali                   |
| Operating System | Kali GNU/Linux Rolling |
| Agent Version    | 4.14.5                 |
| Agent ID         | 003                    |

---

# Step 1 - Verify Connectivity

Before installation, connectivity between Kali and the Wazuh Manager was verified.

### Command

Run on: Kali Linux

```bash
ping -c 4 192.168.21.135
```

### Result

```text
4 packets transmitted
4 packets received
0% packet loss
```

### Purpose

Confirms network connectivity between the endpoint and the manager.

---

# Step 2 - Verify Agent Status

The system was checked for an existing Wazuh agent installation.

### Command

Run on: Kali Linux

```bash
sudo systemctl status wazuh-agent
```

### Result

```text
Unit wazuh-agent.service could not be found.
```

### Finding

The Wazuh agent was not installed.

---

# Step 3 - Add Wazuh Repository

### Import Repository Key

Run on: Kali Linux

```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh.gpg
```

### Add Repository

Run on: Kali Linux

```bash
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
```

### Update Package Index

Run on: Kali Linux

```bash
sudo apt update
```

### Result

Repository synchronization completed successfully.

---

# Step 4 - Install Wazuh Agent

### Command

Run on: Kali Linux

```bash
sudo apt install wazuh-agent -y
```

### Installed Version

```text
wazuh-agent 4.14.5-1
```

### Verification

Run on: Kali Linux

```bash
sudo systemctl status wazuh-agent
```

### Result

```text
Loaded: loaded
Active: inactive (dead)
```

### Finding

Installation succeeded but the agent was not yet configured.

---

# Step 5 - Configure Manager Address

The default placeholder value was replaced with the actual Wazuh Manager IP.

### Backup Configuration

Run on: Kali Linux

```bash
sudo cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.bak
```

### Edit Configuration

Run on: Kali Linux

```bash
sudo nano /var/ossec/etc/ossec.conf
```

### Original Configuration

```xml
<address>MANAGER_IP</address>
```

### Updated Configuration

```xml
<address>192.168.21.135</address>
```

### Verification

Run on: Kali Linux

```bash
sudo grep -A 5 "<client>" /var/ossec/etc/ossec.conf
```

### Result

```xml
<client>
  <server>
    <address>192.168.21.135</address>
    <port>1514</port>
    <protocol>tcp</protocol>
  </server>
</client>
```

---

# Step 6 - Enroll Kali Agent

The agent was registered with the Wazuh Manager.

### Command

Run on: Kali Linux

```bash
sudo /var/ossec/bin/agent-auth -m 192.168.21.135
```

### Result

```text
INFO: Requesting a key from server: 192.168.21.135
INFO: Using agent name as: kali
INFO: Waiting for server reply
INFO: Valid key received
```

### Outcome

The manager generated and provided a valid authentication key to the Kali endpoint.

---

# Step 7 - Enable and Start Agent

### Enable Automatic Startup

Run on: Kali Linux

```bash
sudo systemctl enable wazuh-agent
```

### Start Service

Run on: Kali Linux

```bash
sudo systemctl start wazuh-agent
```

### Verify Status

Run on: Kali Linux

```bash
sudo systemctl status wazuh-agent
```

### Result

```text
Active: active (running)
```

### Outcome

The agent was successfully started and configured to start automatically after reboot.

---

# Step 8 - Verify Agent Logs

### Command

Run on: Kali Linux

```bash
sudo tail -30 /var/ossec/logs/ossec.log
```

### Observed Activity

```text
Starting Security Configuration Assessment scan
Monitoring journal entries
File integrity monitoring scan ended
Rootcheck scan completed
```

### Outcome

The agent successfully began collecting and processing telemetry.

---

# Step 9 - Verify Dashboard Registration

### Dashboard Path

```text
Endpoints
```

### Verification Results

| Agent ID | Hostname       | Status |
| -------- | -------------- | ------ |
| 002      | WIN10-ENDPOINT | Active |
| 003      | kali           | Active |

### Dashboard Summary

```text
Active Agents: 2
Disconnected: 0
Pending: 0
Never Connected: 0
```

---

# Final Architecture

```text
Ubuntu Server (192.168.21.135)
│
├── Wazuh Manager
├── Wazuh API
├── Wazuh Dashboard
│
├── Agent #002
│   └── Windows 10 Endpoint
│
└── Agent #003
    └── Kali Linux 2025.4
```

---

# Lessons Learned

* Wazuh agents require manager IP configuration before enrollment.
* Agent enrollment is performed using `agent-auth`.
* Successful enrollment results in a unique authentication key.
* Installing an agent does not automatically start telemetry collection.
* Services must be enabled and started manually after installation.
* Dashboard verification should always be performed after enrollment.
* Multiple operating systems can be monitored by a single Wazuh Manager.

---

# Skills Demonstrated

* Linux system administration
* Wazuh agent deployment
* Agent enrollment and authentication
* Service management using systemd
* Configuration management
* Endpoint onboarding
* SIEM infrastructure management
* Security monitoring validation

---

## Conclusion

Kali Linux was successfully enrolled into the Wazuh environment as Agent ID 003. The endpoint is actively communicating with the Wazuh Manager and forwarding telemetry for security monitoring and detection use cases.
