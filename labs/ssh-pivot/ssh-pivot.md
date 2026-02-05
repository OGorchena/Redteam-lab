# SSH Pivot Lab — File Transfer + Tunneling (Host ↔ VM)

## 🎯 Goal
Understand SSH as a **process-backed encrypted TCP pipe** and use it for:
- interactive access
- file transfer
- tunneling / pivoting (L/R/D)

Focus: mechanics + red-team mindset + diagnostics (not memorizing commands).

Based on notes: SSH Pivot Notes (Manjaro host ↔ Kali VM).

---

## 🧪 Lab Setup (Roles)
- **Manjaro (host)** — attacker / operator workstation
- **Kali (VM)** — target / pivot box

SSH model:
> Tunnel exists ⇔ ssh process lives  
> Port exists ⇔ process listens (LISTEN)

---

## ✅ Prerequisites

### Install + start sshd
Manjaro (server side when you want to SSH into Manjaro):
```bash
sudo pacman -S openssh
sudo systemctl enable --now sshd
```

Kali (server side when you want to SSH into Kali VM):
```bash
sudo apt update
sudo apt install -y openssh-server
sudo systemctl enable --now ssh
```

### Quick health checks
```bash
ip a
ping -c 1 <peer_ip>
ss -lntp | grep ':22'
systemctl status sshd  # Manjaro
systemctl status ssh   # Kali
```

---

## 🧩 Lab 0 — Mental Model (must know)
SSH = encrypted TCP pipe.  
If ssh process dies → pipe dies → tunnels die.

Truth sources:
```bash
ss -lntp | grep PORT
lsof -iTCP:PORT
ps aux | grep ssh
```

---

## 🔐 Lab 1 — Key-based SSH (no passwords)

### 1) Create key (on client)
```bash
ssh-keygen -t ed25519 -a 64
```

### 2) Copy key to server
```bash
ssh-copy-id user@server
```

### 3) Fix permissions (server)
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

✅ Success criteria:
- `ssh user@server` logs in **without** password

If password still asked:
- wrong user / wrong key / key not in `authorized_keys` / permissions broken.

---

## 🧷 Lab 2 — Host key / known_hosts (MITM hygiene)
Connect first time:
```bash
ssh user@server
```

Understand:
- server host key is stored in `~/.ssh/known_hosts`
- blind “yes” = MITM risk

Task:
- inspect entries:
```bash
cat ~/.ssh/known_hosts | head
```

---

## 🧠 Lab 3 — SSH Agent + Forwarding (risk-aware)

### Agent
```bash
eval $(ssh-agent)
ssh-add ~/.ssh/id_ed25519
ssh-add -l
```

### Forwarding
```bash
ssh -A user@jumpbox
```

Task:
- Explain why `-A` is powerful and why it’s risky.

✅ Success criteria:
- you understand: compromised jumpbox can abuse your agent.

---

## 📦 Lab 4 — File Transfer (3 methods)

### scp
```bash
scp file.txt user@host:/tmp/
scp user@host:/tmp/file.txt .
```

### sftp (interactive)
```bash
sftp user@host
```

### python http.server (lab only)
On sender:
```bash
python3 -m http.server 8000
```

On receiver:
```bash
curl http://<ip>:8000/file.txt -o file.txt
```

✅ Success criteria:
- you can move a file both directions and explain tradeoffs.

---

## 🧱 Lab 5 — Local Port Forward (-L)
Scenario: You want to access a service on the remote side via your localhost.

Example:
```bash
ssh -L 8080:127.0.0.1:8000 user@host
```

Checks:
```bash
ss -lntp | grep 8080
curl http://127.0.0.1:8080
```

✅ Success criteria:
- Local port opens only while SSH process is alive.

---

## 🪝 Lab 6 — Remote Port Forward (-R)
Scenario: Expose your local service to the remote host.

Example:
```bash
ssh -R 2222:127.0.0.1:22 user@host
```

Checks (on remote host):
```bash
ss -lntp | grep 2222
ssh -p 2222 user@127.0.0.1
```

✅ Success criteria:
- remote port exists only while SSH process is alive.

---

## 🧦 Lab 7 — Dynamic SOCKS Proxy (-D) (Pivot)
Create SOCKS proxy:
```bash
ssh -D 1080 user@host
```

Test via curl:
```bash
curl --proxy socks5h://127.0.0.1:1080 http://example.com
```

Checks:
```bash
ss -lntp | grep 1080
```

✅ Success criteria:
- traffic can be proxied through SOCKS while SSH process lives.

---

## 🧯 Lab 8 — Process lifecycle (Ctrl+C vs exit)
- Ctrl+C kills the **foreground process**
- exit closes shell and kills dependent processes

Tasks:
1) Start a tunnel, confirm LISTEN port
2) Kill the SSH process
3) Confirm port is gone

Truth commands:
```bash
ss -lntp | grep PORT
ps aux | grep ssh
```

---

## 🧨 Lab 9 — “Address already in use”
When port is already bound:
```bash
ss -lntp | grep PORT
kill <PID>
```

✅ Success criteria:
- you can identify which process owns the port and free it.

---

## 🧭 Diagnostics Playbook (repeat every time)
1) Network path (IP, route, ping)
2) Service (sshd running? port listening?)
3) SSH model (process ↔ port ↔ tunnel)

---

## 🟥 Red Team Notes (Key Takeaways)
- `authorized_keys` can act as persistence (on real targets: high impact)
- agent forwarding can enable lateral movement (also a risk)
- process ↔ port relationship is the foundation of all tunneling/pivoting

---

## ✅ Completion Checklist
- [ ] Key-based SSH works (no password)
- [ ] You can explain host key / known_hosts and MITM risk
- [ ] Agent is used and you can explain `-A` risk
- [ ] scp + sftp + http.server used at least once
- [ ] You created and verified -L, -R, -D tunnels
- [ ] You proved “tunnel exists ⇔ ssh process lives” using `ss` / `ps`
