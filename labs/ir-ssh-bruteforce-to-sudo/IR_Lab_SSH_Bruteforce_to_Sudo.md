# IR Lab: SSH Brute Force → Credential Compromise → Sudo Activity (Ubuntu Victim)

> **Goal:** generate an incident (Red) and investigate it (Blue) using **SSH logs + sudo logs + auditd**.  
> **Lab-only:** run these steps only in your own VMs / authorized environment.

---

## Environment

### Roles
- **Attacker:** Kali Linux (Hydra)
- **Victim:** Ubuntu Server 22.04 LTS (sshd + auditd)
- **Admin workstation (optional):** Manjaro host SSH’ing into Ubuntu (for copy/paste convenience)

### Example IPs (my lab)
- **Ubuntu victim:** `192.168.50.76`
- **Kali attacker:** `192.168.50.186`
- **Manjaro host (legit admin):** `192.168.50.30`

---

## Requirements
- VirtualBox
- Kali VM + Ubuntu Server VM
- ~16GB RAM recommended

---

## Network model

### Recommended (safer)
- Kali: **Adapter1 NAT** + **Adapter2 Host-Only**
- Ubuntu: **Host-Only**

### What I used (works, but less isolated)
- Kali: **Bridged**
- Ubuntu: **Bridged**

> Bridged puts VMs into your real LAN. It’s ok for learning, but Host-Only is safer for a hacking lab.

---

## Victim minimal setup (Ubuntu)

### Install and start SSH + auditd
```bash
sudo apt update
sudo apt install -y openssh-server auditd audispd-plugins
sudo systemctl enable --now ssh
sudo systemctl enable --now auditd
```

### (Optional) Make SSH logs more verbose
Edit `/etc/ssh/sshd_config`:
```text
LogLevel VERBOSE
```

Restart SSH:
```bash
sudo systemctl restart ssh
```

---

## Attacker setup (Kali)

Create a small password list (lab-only):
```bash
printf "123456\npassword\nqwerty\nletmein\nadmin\n" > pw.txt
```

---

## SSH terminal error: `Error opening terminal: xterm-kitty`

### Problem
When SSH’ing from Manjaro using **kitty**, some `ncurses` apps fail (e.g., `nano`, `htop`, `clear`) with:
```text
Error opening terminal: xterm-kitty
```

### Root cause (short)
- kitty sets `TERM=xterm-kitty`
- SSH forwards `TERM` to the server
- Ubuntu server has no `terminfo` entry for `xterm-kitty`
- `ncurses` programs can’t determine terminal capabilities → crash

### Fix (quick, per-session)
On Ubuntu (inside the SSH session):
```bash
export TERM=xterm-256color
```

### Fix (persistent)
```bash
echo 'export TERM=xterm-256color' >> ~/.bashrc
source ~/.bashrc
```

---

## Pre-flight connectivity checks (before any attack)

From Kali → Ubuntu:
```bash
ping -c 2 192.168.50.76
nc -vz 192.168.50.76 22
ssh victim@192.168.50.76
```

On Ubuntu:
```bash
systemctl status ssh --no-pager
sudo ss -tulpn | grep :22
```

---

## Scenario A — SSH brute force (no compromise)

### Run Hydra (Kali)
```bash
hydra -l victim -P pw.txt ssh://192.168.50.76 -t 4
```

| Option | Meaning |
|---|---|
| `hydra` | brute-force tool |
| `-l victim` | username |
| `-P pw.txt` | password list |
| `ssh://192.168.50.76` | target SSH |
| `-t 4` | 4 parallel tasks |

Expected output:
- `0 valid password found`

### IR: confirm brute force in victim logs

SSH logs (journal):
```bash
sudo journalctl -u ssh --since "10 minutes ago" --no-pager
```

Count failed attempts by source IP (journal):
```bash
sudo journalctl -u ssh --since "10 minutes ago" \
  | grep "Failed password" \
  | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr | head
```

Check if any successful login happened:
```bash
sudo journalctl -u ssh --since "30 minutes ago" | grep "Accepted" || true
```

---

## Scenario B — Credential compromise + sudo activity (controlled)

> Simulates: brute force → successful SSH login → sudo execution → artifact creation.

### Step 1: Set a weak lab password (Ubuntu)
Set `victim` password to something that exists in `pw.txt` (example: `admin`):
```bash
sudo passwd victim
```

### Step 2: Run Hydra again (Kali)
```bash
hydra -l victim -P pw.txt ssh://192.168.50.76 -t 4
```

Expected:
- Hydra prints the valid password found.

### Step 3: Log in and do minimal post-compromise actions
```bash
ssh victim@192.168.50.76
whoami
id
sudo -l
sudo id
touch /tmp/comprom_by_kali
exit
```

---

## IR Investigation (Scenario B)

### 1) Find the compromise moment (successful SSH auth)
```bash
sudo journalctl -u ssh --since "30 minutes ago" --no-pager \
  | grep -E "Failed password|Accepted|session opened"
```

**Note about multiple “Accepted” lines:**  
You may see logins from **Kali attacker IP** (`192.168.50.186`) and also from your **Manjaro admin IP** (`192.168.50.30`). IR must distinguish attacker activity vs legit administration.

### 2) Confirm sudo usage (what ran as root)
```bash
sudo journalctl --since "30 minutes ago" --no-pager | grep sudo
```

Look for fields like:
- `USER=root`
- `COMMAND=/usr/bin/id` (or other command)
- `PWD=/home/victim`
- `TTY=pts/X`

### 3) Find artifact creation without knowing the filename (time-based hunting)
Use the time window after the compromise:
```bash
sudo find /tmp -type f -newermt "2026-02-18 11:41" -ls
```

---

## Cleanup (after lab)
```bash
sudo rm -f /tmp/comprom_by_kali
sudo passwd victim
```
