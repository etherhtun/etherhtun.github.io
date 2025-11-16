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


# Lesson 26: Securing RHEL with SELinux (RHCSA Focus)

## 26.1 Understanding the Need for SELinux
SELinux (Security-Enhanced Linux) is a mandatory access control (MAC) system that adds an additional security layer on top of traditional Linux permissions.  
It prevents services from accessing files or resources they are not explicitly allowed to access.

Benefits:
- Protects against 0-day vulnerabilities
- Limits damage from compromised services
- Enforces strict access rules using security contexts

---

## 26.2 Using SELinux Modes

Check current SELinux mode:
```bash
getenforce
```

Modes:
- **Enforcing** — SELinux policies are applied and violations are blocked  
- **Permissive** — Violations are logged but not blocked (used for troubleshooting)  
- **Disabled** — SELinux is off  

Temporarily switch modes:
```bash
setenforce 0      # Permissive
setenforce 1      # Enforcing
```

Permanent configuration:
```
/etc/selinux/config
```

---

## 26.3 Exploring SELinux Components

SELinux is built around:
- **Policies** — define allowed actions  
- **Contexts** — labels assigned to files, processes, ports  
- **Booleans** — toggles that enable/disable rules  
- **Log messages** — violations recorded in `/var/log/audit/audit.log`  

A typical SELinux context has 4 parts:
```
user:role:type:level
```

Example:
```
system_u:object_r:httpd_sys_content_t:s0
```

---

## 26.4 Using File Context Labels

Show file SELinux contexts:
```bash
ls -Z
```

Show process contexts:
```bash
ps -eZ
```

Change file context temporarily (not persistent):
```bash
chcon -t httpd_sys_content_t /var/www/html/index.html
```

Restore default contexts:
```bash
restorecon -Rv /var/www/html
```

---

## 26.5 Finding the Right Context

Use `semanage fcontext` to check and assign correct contexts.

View existing rules:
```bash
semanage fcontext -l
```

Add a persistent rule for custom directory:
```bash
semanage fcontext -a -t httpd_sys_content_t "/myweb(/.*)?"
restorecon -Rv /myweb
```

---

## 26.6 Managing SELinux Port Access

View allowed ports:
```bash
semanage port -l | grep http
```

Add new allowed port:
```bash
semanage port -a -t http_port_t -p tcp 8080
```

Remove port rule:
```bash
semanage port -d -t http_port_t -p tcp 8080
```

This is required if you run services on non-standard ports.

---

## 26.7 Using SELinux Booleans

List all SELinux Booleans:
```bash
getsebool -a
```

Enable a Boolean temporarily:
```bash
setsebool httpd_use_nfs on
```

Enable permanently:
```bash
setsebool -P httpd_use_nfs on
```

Useful examples:
- `httpd_can_network_connect` — allow Apache outbound connections  
- `samba_enable_home_dirs` — allow Samba to use home directories  

---

## 26.8 Analyzing SELinux Log Messages

SELinux denial logs:
```
/var/log/audit/audit.log
```

Search for denials:
```bash
grep "denied" /var/log/audit/audit.log
```

Use `ausearch`:
```bash
ausearch -m AVC -ts recent
```

Human-readable log interpretation:
```bash
sealert -a /var/log/audit/audit.log
```

---

## 26.9 Troubleshooting SELinux

Steps:

### 1. Check logs for denials  
```bash
ausearch -m AVC -ts recent
```

### 2. Verify file context  
```bash
ls -Z /path
restorecon -Rv /path
```

### 3. Check if a Boolean is required  
```bash
getsebool -a | grep http
```

### 4. Check port context  
```bash
semanage port -l | grep <port>
```

### 5. Switch to Permissive mode temporarily  
```bash
setenforce 0
```

Return to Enforcing:
```bash
setenforce 1
```

---

# RHCSA Exam Focus Points

You must know how to:

- Switch SELinux modes
- Read SELinux contexts
- Apply correct file context with `restorecon`
- Add persistent context rules with `semanage fcontext`
- Add port rules using `semanage port`
- Enable and disable Booleans
- Troubleshoot AVC denials via logs

