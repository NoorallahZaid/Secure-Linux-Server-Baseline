# Linux Server Hardening

A hands-on Linux security hardening project focused on strengthening SSH access, enforcing least-privilege permissions, protecting against brute-force attacks, monitoring security events with Wazuh, and establishing system rollback and recovery procedures.

The project follows a structured **Audit → Harden → Verify → Recover** lifecycle and evaluates security improvements using Lynis and Nmap.

## Key Features

* **SSH Hardening** — Secured OpenSSH through non-default port configuration, disabled root login, restricted authentication attempts, and enforced key-based authentication.
* **User & Access Control** — Applied least-privilege principles through dedicated administrative accounts, account management, and filesystem permissions.
* **Brute-Force Protection** — Configured Fail2Ban to automatically block IP addresses after repeated failed SSH authentication attempts.
* **Security Monitoring** — Deployed Wazuh for centralized security monitoring, log analysis, and dashboard visualization.
* **Security Assessment** — Used Lynis and Nmap to perform pre- and post-hardening assessments.
* **Recovery & Rollback** — Implemented SSH configuration backups and Timeshift snapshots to support recovery from configuration errors.

## Hardening Lifecycle

The project follows a four-phase security hardening lifecycle:

```text
┌──────────┐
│  AUDIT   │  Assess the system and identify security weaknesses
└────┬─────┘
     ↓
┌──────────┐
│  HARDEN  │  Apply security configurations and access controls
└────┬─────┘
     ↓
┌──────────┐
│  VERIFY  │  Re-scan and validate implemented security measures
└────┬─────┘
     ↓
┌──────────┐
│ RECOVER  │  Maintain backups and establish rollback procedures
└──────────┘
```

## Tools & Technologies

| Tool               | Purpose                                          |
| ------------------ | ------------------------------------------------ |
| **Kali Linux**     | Hardened server environment                      |
| **Lynis**          | Security auditing and hardening assessment       |
| **Nmap**           | Port scanning and service discovery              |
| **OpenSSH**        | Secure remote access                             |
| **Fail2Ban**       | Brute-force detection and IP blocking            |
| **Wazuh**          | Security monitoring and log analysis             |
| **Docker**         | Wazuh container deployment                       |
| **Timeshift**      | System snapshots and recovery                    |
| **UFW / iptables** | Firewall and network access control              |
| **Bash**           | System administration and security configuration |

---

# Implementation

## 1. Baseline Assessment

The system was first assessed to establish a security baseline and identify exposed services.

### Lynis Security Audit

```bash
sudo lynis audit system
```

### Service & Port Discovery

```bash
nmap -sV localhost
```

The initial security assessment produced a **Lynis hardening index of 59%**. Nmap was also used to identify exposed ports and running services before applying hardening measures.

---

## 2. SSH Hardening

SSH was hardened as a primary remote-access entry point.

### Backup Configuration

The original SSH configuration was backed up before making changes:

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```

### Configuration Changes

The following SSH settings were applied:

| Setting                   | Configuration               | Purpose                                   |
| ------------------------- | --------------------------- | ----------------------------------------- |
| SSH Port                  | `Port 2222`                 | Reduce exposure to automated scans        |
| Root Login                | `PermitRootLogin no`        | Prevent direct root access                |
| Authentication Attempts   | `MaxAuthTries 3`            | Limit repeated authentication attempts    |
| Public Key Authentication | `PubkeyAuthentication yes`  | Enable key-based authentication           |
| Password Authentication   | `PasswordAuthentication no` | Disable password-based SSH authentication |
| X11 Forwarding            | `X11Forwarding no`          | Disable unnecessary X11 forwarding        |

The SSH service was restarted after configuration:

```bash
sudo systemctl restart ssh
```

The configuration was subsequently verified by scanning the system and testing SSH connectivity on the new port.

---

## 3. Fail2Ban Brute-Force Protection

Fail2Ban was configured to monitor authentication failures and automatically block source IP addresses after repeated unsuccessful login attempts.

### Installation

```bash
sudo apt install fail2ban
```

### Configuration

The local jail configuration was created in:

```text
/etc/fail2ban/jail.local
```

Key settings included:

| Parameter                |          Value |
| ------------------------ | -------------: |
| SSH Port                 |         `2222` |
| Maximum Retries          |            `3` |
| Ban Duration             | `3600` seconds |
| Failure Detection Window |  `600` seconds |

### Verification

```bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

This provided automated protection against repeated SSH authentication attempts.

---

## 4. User & Access Control

User and filesystem permissions were configured according to the **principle of least privilege**.

### Administrative User

A dedicated administrative account was created:

```bash
sudo useradd -aG sudo secadmin
sudo passwd secadmin
```

Inactive accounts could be locked when no longer required:

```bash
sudo passwd -l username
```

### Filesystem Permissions

Sensitive directories were assigned restrictive permissions:

| Directory            | Permissions | Purpose                                 |
| -------------------- | ----------: | --------------------------------------- |
| `/srv/seclab/admin`  |       `700` | Owner-only access                       |
| `/srv/seclab/shared` |       `755` | Owner write access; others read/execute |
| `/srv/seclab/logs`   |       `750` | Owner full access; group read/execute   |

Example:

```bash
chmod 700 /srv/seclab/admin
chmod 755 /srv/seclab/shared
chmod 750 /srv/seclab/logs

chown secadmin:secadmin /srv/seclab/admin
```

These controls reduce unnecessary access to administrative and system data.

---

## 5. Wazuh Security Monitoring

Wazuh was deployed to provide centralized security monitoring and log analysis.

Docker was used to deploy the Wazuh environment.

### Docker Setup

```bash
sudo apt install docker.io docker-compose -y
sudo systemctl enable docker --now
```

### Wazuh Deployment

The Wazuh Docker repository was cloned and the required certificates and services were initialized:

```bash
git clone https://github.com/wazuh/wazuh-docker.git -b v4.7.0
cd wazuh-docker

docker-compose -f generate-indexer-certs.yml run --rm generator
docker-compose up -d
```

The Wazuh dashboard was then used for security event and log monitoring.

A Wazuh agent was configured on the system to forward relevant security events to the monitoring infrastructure.

---

## 6. Post-Hardening Evaluation

After applying the hardening measures, the system was reassessed.

### Lynis

```bash
sudo lynis audit system
```

### Nmap

```bash
nmap -sV localhost
```

### Fail2Ban

```bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

The results were compared against the initial baseline.

---

# Results

## Lynis Hardening Index

| Metric           |                   Result |
| ---------------- | -----------------------: |
| Before Hardening |                  **59%** |
| After Hardening  |                  **67%** |
| Improvement      | **+8 percentage points** |

The post-hardening assessment showed an **8-percentage-point increase in the Lynis hardening index**.

## Port Exposure

| Service              | Before         | After            |
| -------------------- | -------------- | ---------------- |
| SSH                  | Port `22` open | Port `2222` open |
| Port 22              | Open           | Closed           |
| Unnecessary Services | Multiple       | Reduced          |

## Brute-Force Protection

| Configuration          |         Result |
| ---------------------- | -------------: |
| Maximum Login Attempts |            `3` |
| Ban Duration           | `3600` seconds |
| Failure Window         |  `600` seconds |
| Automatic IP Blocking  |    **Enabled** |

The combination of SSH hardening and Fail2Ban provided multiple layers of protection against unauthorized remote access.

---

# Rollback & Recovery

Security hardening can introduce configuration errors or, in the case of SSH, potentially cause remote lockout. Recovery mechanisms were therefore included as part of the hardening process.

## SSH Configuration Backup

The original SSH configuration can be restored using:

```bash
sudo cp /etc/ssh/sshd_config.bak /etc/ssh/sshd_config
sudo systemctl restart ssh
```

SSH connectivity should then be verified after restoration.

## Timeshift Snapshots

System snapshots were created before and after major hardening stages.

### Create Snapshot

```bash
sudo timeshift --create --comments "Before hardening"
```

### List Snapshots

```bash
sudo timeshift --list
```

### Restore Snapshot

```bash
sudo timeshift --restore --snapshot '2026-04-20_10-00-01'
```

Snapshots provide an additional recovery layer if configuration-level restoration is insufficient.

## Recovery Procedure

1. Identify the issue using system logs and service status.
2. Restore the affected configuration from backup.
3. Restart the affected service.
4. If necessary, restore the system using a Timeshift snapshot.
5. Verify system functionality and security configuration.

---

# Security Principles Applied

### Defense in Depth

Multiple security controls were implemented rather than relying on a single protection mechanism:

**SSH Hardening → Access Control → Fail2Ban → Firewall → Wazuh Monitoring**

### Least Privilege

User accounts and filesystem permissions were configured to provide only the access required for administrative and operational tasks.

### Attack Surface Reduction

Unnecessary exposure was reduced through service discovery, SSH configuration changes, and network access controls.

### Continuous Monitoring

Wazuh was introduced to provide centralized security event monitoring and log analysis.

### Measurable Security

Lynis and Nmap provided objective pre- and post-hardening assessments, allowing security improvements to be compared against the initial baseline.

### Recoverability

Configuration backups and system snapshots were incorporated to ensure that security changes could be reversed if necessary.

---

# Conclusion

This project demonstrates a structured approach to Linux server hardening by combining system auditing, SSH security, access control, brute-force protection, security monitoring, and recovery planning.

The hardening process increased the measured **Lynis hardening index from 59% to 67%**, while reducing unnecessary service exposure and introducing automated protection and monitoring mechanisms.

The project demonstrates practical experience with **Linux administration, Bash, SSH, network security, security auditing, access control, Docker, Fail2Ban, Wazuh, and system recovery**.
