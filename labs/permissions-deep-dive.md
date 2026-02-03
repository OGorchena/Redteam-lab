# Permissions Deep Dive — Pentest Relevant  
**Theory + Practice (Single Flow)**

---

## 🎯 Goal

Learn how to identify and exploit permission misconfigurations by understanding
**execution context**, not just file permissions.

The practice is designed as a **safe misconfiguration sandbox** that closely
resembles real-world pentest scenarios.

---

## 🧠 Core Idea

In Linux, permissions are evaluated based on **process context**, not intent.

Key questions:
- Who executes the process?
- With which user and groups?
- What capabilities does the tool provide?

Privilege escalation is a consequence of **context abuse**, not commands.

---

## 🧪 Environment

- **OS:** Linux  
- **Scope:** local VM / test system  
- **Mode:** controlled misconfiguration sandbox  
- **Purpose:** educational use only  

---

## 👤 Users & Groups (Foundation)

Every Linux process runs as a user and a set of groups.  
Permissions are evaluated based on **effective identity**, not who logged in.

### Commands

```bash
whoami
id
groups
```

### Pentest Relevance

Focus on privileged or sensitive groups:
- `sudo`
- `docker`
- `lxd`
- `adm`
- `systemd-journal`

Misalignment between user identity and group capabilities
often leads to privilege escalation paths.

---

## 🔑 Sudo Enumeration

### Primary Command

```bash
sudo -l
```

This is one of the most important enumeration steps.

### Key Insight

Risk is defined **not by sudo itself**, but by the **type of allowed command**.

High-risk categories:
- Editors (`vim`, `nano`)
- Pagers (`less`, `more`)
- Interpreters (`python`, `perl`, `awk`)
- Tools with hooks or shell escapes (`tar`, `find`, `tcpdump`)

If a tool is interactive or interprets input,
it often provides a path to command execution.

---

## 🧠 Vim / Less as a Mental Model

Example:

```bash
sudo vim
```

The editor is executed as **root**, even if it is not a shell.

Inside Vim:
- `:ls` — internal Vim command (buffers)
- `:!ls` — execute shell command
- `!` — boundary between application and OS

**Key Insight:**  
If `:!id` returns `uid=0`, there is no need to write files or drop scripts.
Command execution is already achieved.

---

## 🔐 SUID / SGID Concept

- **SUID** — binary executes with the effective UID of its owner  
- **SGID** — process inherits the group of the binary  

Example:

```bash
-rwsr-xr-x root root /usr/bin/passwd
```

Discovery:

```bash
find / -perm -4000 -type f 2>/dev/null
```

### Pentest Relevance

Not every SUID binary is exploitable.  
What matters is **functionality**, such as:
- command execution
- file read/write
- shell access
- unsafe input handling

---

## 📂 Writable Paths & Directories

```bash
find / -writable -type d 2>/dev/null
```

Writable does **not** automatically mean exploitable.

Value exists only if:
- a privileged process executes files from that directory
- the directory is part of `PATH`
- IPC, sockets, or session files are involved

`/tmp` and `/dev/shm` alone are not vulnerabilities.

---

## 🧨 PATH Hijacking Conditions

All conditions must be met:
1. Writable directory exists in `PATH`
2. A privileged process relies on `PATH`
3. The binary is executed without an absolute path

If the attacker launches the command manually,
the vector does not apply.

---

## 🧪 Practice — Misconfiguration Sandbox (Safe)

### 1. Create attacker user

```bash
useradd attacker
passwd attacker
```

### 2. Add dangerous sudo rule

```bash
visudo
attacker ALL=(ALL) NOPASSWD: /usr/bin/vim
```

### 3. Switch user

```bash
su attacker
```

### 4. Enumeration

```bash
id
sudo -l
```

### 5. Exploitation

```bash
sudo vim
:!id
```

Confirm `uid=0`, then execute additional commands
(e.g. reading restricted directories or logs).

---

## 🧠 Misconfiguration Analysis

The issue is **not sudo itself**.

Root causes:
- misunderstanding tool capabilities
- convenience over security
- legacy configurations and technical debt

The tool was never designed to be a privilege boundary.

---

## 🧠 Observations, Conclusions & Limitations

- Privilege escalation is about **context**, not commands  
- Interactive tools are common escalation vectors  
- SUID and sudo risks depend on **behavior**, not flags  

Understanding execution boundaries makes exploit paths obvious.

All actions were performed in a controlled misconfiguration sandbox
for educational purposes only.

---

## 🧠 Final Takeaway

Privilege escalation is not about tricks or magic commands.  
It is about understanding:
- execution context
- tool behavior
- boundaries between user, process, and system

