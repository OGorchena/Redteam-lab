# IR Lab — SSH Brute Force → Credential Compromise → Sudo Escalation

## 🎯 Objective

Simulate and investigate a realistic attack chain:

**Brute force → Successful SSH login → Privilege escalation via sudo**

This lab combines Red Team execution with Blue Team investigation to practice:

- SSH brute-force detection  
- Authentication validation  
- Source IP attribution  
- Sudo activity analysis  
- Timeline reconstruction  
- Evidence collection  

---

# 🏗 Lab Architecture

## Roles

- **Attacker:** Kali Linux (Hydra)
- **Victim:** Ubuntu Server 22.04 (sshd + auditd)
- *(Optional)* Admin workstation: Manjaro

---

# 📦 Requirements

## Host
- VirtualBox
- 16GB RAM recommended
- 60–100GB disk space

## VMs
- Kali Linux
- Ubuntu Server (victim)

---

# 🌐 Network Model

## Recommended (Isolated)
- Kali: NAT + Host-Only  
- Ubuntu: Host-Only  

> Host-Only is safer to avoid exposing lab attacks to your real network.

---

# 🛠 Victim Setup (Ubuntu)

## Install services

```bash
sudo apt update
sudo apt install -y openssh-server auditd audispd-plugins
sudo systemctl enable --now ssh
sudo systemctl enable --now auditd
