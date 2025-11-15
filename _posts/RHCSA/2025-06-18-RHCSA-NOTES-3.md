---
title: "RHCSA NOTES 2"
date: 2025-06-18 00:00:00 +08:00
categories: [RHCSA]
tags: [RHCSA]
---
# Lesson 13 Monitoring Activity

- Managing Shell jobs 
    - sleep 1000 &                  # & is running in background process 
    - bg                            # show backgrounds  process 
    - jobs                          # current running jobs with PID 
    - fg                            # change from backgroud to foreground process 

- Handling Process, CPU & Memory  
    - ps aux | less  
    - ps -fax                       # hierarchical process  
    - ps -fU linda                  # show processes owned by linda 
    - ps -d --forest -C sshd
    - ps -eo pid,ppid,user,cmd      # custom show 
    
    - free -m                       # memory status
    - less /proc/meminfo  

    - lscpu 
    - uptime 
    - dd if=/dev/zero of=/dev/null &   # create process to overload the memory
    - while true; do :; done & 
    - cat /dev/zero > /dev/null &
    - sleep infinity &
    - yes | dd of=/dev/null &
    - dd if=/dev/random of=/dev/null bs=1 count=0 &
    - top 
[outputs](/Documents/RHCSA/Lesson13-Monitoring-Activity.txt)   


## Managing processes 

- man 7 signals 
- ps aux | grep defunct                 # find the zombie process 
- ps fax | grep defunct                 # find process forest 


### Managinf process prioriy

- nice -n 19 dd if=/dev/zero of=/dev/null &     # set priorty 
- nice -n -19 dd if=/dev/zero of=/dev/null &    #set low priorty 
- tuned-adm profile 

- ps -u username 
- pkill -u linda 
- loginctl list=users
- loginctl terminate-user


# Lesson 15 - Working with systemd

- Service units to use the start process 
- Socket units monitor activity on a port and start the corresponding service unit when needed  
- Timer units are used to start services periodically 
- Path units can start service units when activity is detected in the file system 


- systemctl -t help              # to check the usages 
- systemctl list-units
- systemctl list-units -t timer 
- sytstectl list-units-files 

- systemctl status sshd 
    - status Active: service status 
    - loaded:  which configration is loaded. 
- systemctl start 
- systemctl stop 
- systemctl enable [--now]
- systemctl disable [--now]
- systemctl reload 
- systemctl restart 


## Systemd file system 

- /usr/lib/systemd/system                   # default file system 
- /etc/systemd/system                       # custom file system 
- systemctl cat httpd.service               # show httpd service 
- systemctl edit httpd.service 

- systemctl list-dependencies               # show dependencies 
- systemctl list-dependencies sshd.service  # check detail dependencies on ssh service 


# lesson 16 Task Scheduling 

- systemd , crond, at  are most common tools to scheduling tasks 
- dnf install sysstat 
- systemctl list-unit-files systat* 
- systemctl cat sysstat-collect.timer 

from pathlib import Path

# Create markdown content for RHCSA Task Scheduling notes
markdown_content = """# RHCSA Lesson: Task Scheduling

## ✅ Objectives
- Automate tasks using **`cron`**, **`at`**, and **`systemd timers`**
- Understand cron syntax and configuration files
- Manage one-time vs recurring jobs

---

## 📆 1. Periodic Task Scheduling with `cron`

### What is `cron`?
A time-based job scheduler in Linux used to run commands or scripts at specific intervals (minutes, hours, days, etc.).

### Key Files:
- User crontabs: `/var/spool/cron/username`
- System-wide crontab: `/etc/crontab`
- Cron job directories: `/etc/cron.d/`, `/etc/cron.daily/`, `/etc/cron.hourly/`, etc.
- Log file: `/var/log/cron`

### Cron Syntax (Fields):
*     *     *     *     *     command
-     -     -     -     -
|     |     |     |     |
|     |     |     |     +----- Day of the week (0–7) (Sun=0 or 7)
|     |     |     +------- Month (1–12)
|     |     +--------- Day of the month (1–31)
|     +----------- Hour (0–23)
+------------- Minute (0–59)


**Run a script every day at 2:30 AM:**

```bash 
30 2 * * * /home/user/backup.sh 
```

### Edit crontab 
```bash 
crontab -e         # Edit current user's crontab
crontab -l         # List user's cron jobs
crontab -r         # Remove current user's crontab
```

**One-time scheduling with "at"**

```bash 
echo "shutdown -h now" | at now + 10 minutes
```

**Systemd Timers**

- Timer: something.timer
- Service: something.service

***1. Create a service file /etc/systemd/system/myjob.service***

```bash 

[Unit]
Description=My periodic job

[Service]
Type=oneshot
ExecStart=/usr/local/bin/myscript.sh

```

***2. Create a timer file /etc/systemd/system/myjob.timer***

```bash 

[Unit]
Description=Run my job every 10 minutes

[Timer]
OnBootSec=10min
OnUnitActiveSec=10min
Unit=myjob.service

[Install]
WantedBy=timers.target

```

***3. Enable and Start*** 

```bash 
systemctl enale --now myjob.timer 
```

***4. Check status***

```bash 
systemctl list-timers 
``` 