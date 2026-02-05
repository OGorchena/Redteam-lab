# SSH Pivot Lab — Index

## 🎯 Goal
Learn SSH tunneling and pivoting as a **process-backed encrypted TCP pipe**:
- Local / Remote port forwarding (`-L`, `-R`)
- Dynamic SOCKS proxy (`-D`)
- Basic file transfer workflows (scp/sftp/http.server)

## 🧪 Setup
- Host: Manjaro (operator)
- VM: Kali (pivot/target)
- Requirement: `sshd` running on the side you connect to

Quick checks:
```bash
ip a
ping -c 1 <peer_ip>
ss -lntp | grep ':22'
systemctl status sshd  # Manjaro
systemctl status ssh   # Kali
```

📄 Lab Content

Full lab instructions: lab.md (in this directory).

✅ Completion Checklist

 SSH key auth works (no password)

 You can explain known_hosts and MITM risk

 You created and verified -L tunnel (ss -lntp)

 You created and verified -R tunnel

 You created and verified SOCKS proxy -D

 You proved: tunnel exists ⇔ ssh process lives
