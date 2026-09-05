# Linux Server Hardening Project
## Overview
This project implements a systematic Linux server hardening methodology on a Kali Linux server to enhance security posture through SSH configuration enforcement, filesystem permission management, automated logging analysis with Wazuh, and a robust rollback and recovery plan.

The research demonstrates that implementing a structured hardening approach can increase the Lynis hardening index from 59% to 67%, significantly reducing the system's vulnerability to attacks.

## Table of Contents
Key Features

Hardening Lifecycle

Tools and Technologies

## Key Features
SSH Hardening: Non-default port configuration, root login disabled, key-based authentication enforced

User Access Control: Least privilege principle implementation, dedicated admin users, strict permissions

Automated Log Analysis: Wazuh integration for centralized security monitoring and dashboard visualization

Brute Force Protection: Fail2Ban configuration for automatic IP blocking after repeated failures

Measurable Security Improvement: Lynis and Nmap tools used for pre/post hardening assessment

Recovery Planning: Timeshift snapshots and configuration backups for reliable rollback

## Hardening Lifecycle
The project follows a 4-phase hardening lifecycle:

Phase A - Audit: System assessment using Lynis, OpenSCAP, or Auditd to identify vulnerabilities

Phase B - Harden: Implementation of security configurations and hardening techniques

Phase C - Verify: Re-auditing to validate security improvements

Phase D - Recover: Backup and recovery planning for system restoration

https://media/image25.png

## Tools and Technologies

| Tool | Purpose |
|------|---------|
| **Lynis** | Security auditing and hardening index scoring |
| **Nmap** | Port scanning and service exposure detection |
| **OpenSSH** | Secure remote access configuration |
| **Fail2Ban** | Log monitoring and brute force protection |
| **Wazuh** | Centralized log analysis and security monitoring |
| **Docker** | Wazuh container deployment |
| **Timeshift** | System snapshot and recovery management |
| **iptables/UFW** | Firewall configuration |
