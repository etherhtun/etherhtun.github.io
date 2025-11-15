---
title: "RHCSA NOTES 3"
date: 2025-11-15 00:00:00 +08:00
categories: [RHCSA]
tags: [RHCSA]
---

# Lesson 24: Managing SSH 

## 24.1 Understanding SSH Key-based Login
SSH key-based login allows you to authenticate without typing a password.

It uses:
- **Private key** → stays on your workstation (`~/.ssh/id_rsa`)
- **Public key** → stored on the server (`~/.ssh/authorized_keys`)

Advantages:
- More secure than passwords
- Automates scripts (Ansible, rsync)
- Required for many DevOps tools

---

## 24.2 Setting up SSH Key-based Login

### Step 1: Create SSH key pair on client
```bash
ssh-keygen -t rsa -b 4096
```

Keys stored at:
```
~/.ssh/id_rsa          (private key)
~/.ssh/id_rsa.pub      (public key)
```

### Step 2: Copy public key to server
```bash
ssh-copy-id user@server
```

Alternative manual method:
```bash
cat ~/.ssh/id_rsa.pub | ssh user@server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

### Step 3: Test login
```bash
ssh user@server
```
You should login without password.

---

## 24.3 Caching SSH Keys (ssh-agent)

Caching lets you unlock your private key once, not every time you SSH.

Start the agent:
```bash
eval $(ssh-agent)
```

Add your private key:
```bash
ssh-add
```

To list cached keys:
```bash
ssh-add -l
```

Useful for:
- rsync
- git
- automation scripts

---

## 24.4 Defining SSH Client Configuration

Edit client config file:
```
~/.ssh/config
```

Example:
```
Host myserver
    HostName 192.168.1.10
    User root
    Port 22
    IdentityFile ~/.ssh/id_rsa
```

Then you can simply run:
```bash
ssh myserver
```

RHCSA exam tip:
You may need to connect to multiple servers efficiently — this helps.

---

## 24.5 Exploring Common SSH Server Options

Edit:
```
/etc/ssh/sshd_config
```

Common settings:

- `Port 22` — Change port for security  
- `PermitRootLogin no` — Disable direct root login  
- `PasswordAuthentication no` — Force key-based login  
- `PubkeyAuthentication yes` — Enable key-based login  
- `AllowUsers user1 user2` — Restrict login to certain users  
- `X11Forwarding no` — Disable GUI forwarding

After editing:
```bash
sudo systemctl restart sshd
sudo systemctl enable sshd
```

Check status:
```bash
systemctl status sshd
```

---

## 24.6 Copying Files Securely (scp)

Copy from local → remote:
```bash
scp file.txt user@server:/path/
```

Copy folder:
```bash
scp -r dir/ user@server:/path/
```

Copy from remote → local:
```bash
scp user@server:/path/file.txt .
```

---

## 24.7 Synchronizing Files Securely (rsync over SSH)

Basic sync:
```bash
rsync -avz file.txt user@server:/path/
```

Sync while deleting removed files:
```bash
rsync -avz --delete /src/ user@server:/dest/
```

Sync directory to remote:
```bash
rsync -avz /var/www/ user@server:/backup/www/
```

rsync uses SSH by default.

---

# RHCSA Exam Focus Points

You must be able to:

- Generate SSH keys
- Configure key-based authentication
- Disable root login
- Modify `sshd_config` and restart service
- Copy files using scp
- Sync directories with rsync
- Use ssh-agent for key caching

Good luck with your lab practice — want practice tasks or a quick quiz next?
