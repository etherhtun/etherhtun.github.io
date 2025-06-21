---
title: "RHCSA NOTES-DASHBOARD"
date: 2025-06-18 00:00:00 +08:00
categories: [RHCSA]
tags: [RHCSA]
---
## Lesson 1 Understanding Red Hat Enterprise Linux

### stream
- Fedora Stream (community project sponsored by Red Hat)
- Centos Stream (platform for RHEL -> Not recommended for production)
- Red Hat Enterprise Linux

## Lesson 2 Installing Red Hat Enterprise Linux

### Partitioning
```
- /boot - 500 Mb
- /     - 12 Gb
- /home - 3 Gb
- swap  - 1 Gb 
```
## Lesson 3 Using the Command Line
- Virtual terminals - Ctrl+Alt+Fn(f1, f2, etc)
- w or who
- chvt (2, 3, etc)

## Lesson 4 Exploring Essential Tools
- man
``` 
  - search / and n
  - man -a passwd
  - sudo mandb -> man -k user | grep
```
- nano myfile
- vim
```
  - Esc
  - i, a
  - yy and p
  - :se number
  - v -> select -> dd
  - u
  - :%s/do/don't/g
  - vimtutor
```
- useradd (anna)
- passwd (anna)
- rm -r (directory)

## Lesson 5 Understanding The Bash Shell
```
- \> : STDOUT
- \>\> : STDOUT APPEND
- 2> /dev/null : STDERR
- \< : STDIN
- PIPPING : STDOUT of first to STDIN to second
- grep
- history
  - !number
  - Ctrl-r
- which
- ls -l $(which passwd)
- env
- echo $PATH
- /etc/profile (generic Bash startup for login shell)
- /etc/bashrc (processed while openig a subshell)
- ~/.bash_profile (user-specifi version of /etc/profile)
- .bashrc (to add alias)
- source .bash_profile (to use update profile)
```
## Lesson 6 Using Essential File Management Tools
```
- man file-hierarchy
- sudo -i
- find / -name "hosts" 2>/dev/null
- find / -type f -size +100M
- mkdir -p /find/contents; find /etc -size +1k -exec grep -l atr {} \; -exec cp {} /find/contents/ \;
- find /etc/ -name "*" | xargs grep "127.0.0.1" 2> /dev/null
- findmnt vs lsblk vs df -h
- /bin - toolbox room (tools like ls, cp)
- /etc - rulebook (configuration files like /etc/passwd)
- /home - family room (perfonal files, data, settings)
- /mnt - guest packing spot (attached temporary filesystems like USB)
- /usr - the library (/usr/bin - user-installed programs)
- /var - mailroom (variable data like log)
- ln /etc/hosts myhosts
- ln -s myhosts sysmhosts
- tar -cvf vs tar -xvf vs tar -tvf
- tar -czvf file.tar.gz (most common)
```
## Lesson 8 - Using root privileges

```bash
su -                  # Switch to root with login shell
su -c 'command'       # Run one root command
sudo command          # Run command as root
sudo -i               # Open root shell
usermod -aG wheel user#Add user into wheel permission to grant root access
visudo                # Safely edit sudoers file
sudo mkdir /etc/sudoers.d/         # Optional config directory
sudo visudo -f /etc/sudoers.d/dev  # Create rule file for user 'dev'
whoami                #Check user  
id                    #check user and group permssion 
groups username       # Check user’s groups



#command restrictions 
#edit /etc/sudoers.d/jane  , edit for jane only 
jane ALL=(ALL) NOPASSWD: /bin/systemctl restart apache2 #jane can restart apache2 service 
jane ALL=/usr/bin/passwd,!/usr/bin/passwd root #jane can reset password for all users but not for root users
jane ALL=/user/sbin/useradd, /usr/sbin/usermod, /user/sbin/userdel, ALL=/usr/bin/passwd,!/usr/bin/passwd root

#check logs 
/var/log/secure 

#Check SSh services 
sudo systemctl status sshd
sudo systemctl restart sshd 
#deny ssh root login 
#"/etc/ssh/sshd_config" 
PermitRootLogin no 
```
# Lesson 9 - Managing Users and Groups

- User properties are managed in ** /etc/passwd **
- User password keep under ** /etc/shadow **  

- Terminology
    - Name : name of the account
    - UID : unique identifier for users
    - GID - group ID
    - Home directory : personal files
    - shell : program that will be started after successful authenticated

```bash
#Add and set password for user F
sudo useradd jane 
sudo passwd jane 
sudo useradd -m -s /bin/bash jane  # -m create home directory, -s create default shell 
sudo useradd lori -u 2000 -s /sbin/nologin  #create user with id 2k and no login access 

#Delete 
sudo usermod -L jane      #lock accout 
sudo usermod -U jane      #Unlock account 
sudo usermod -e 2025-08-01  jane  # set expire 
sudo userdel jane 
sudo userdel -rf jane      #Remove home directory 
sudo usermod -s /sbin/nologin jane  #disable shell access 

```
- User default permissions /etc/login.defs 
    - PASS_MAX_DAYS 
    - PASS_MIN_DAYS
    - PASS WARN AGE 
    - HOME_MODE 

```bash
#create startup folder in new users 
touch /etc/skel/hello     #when newuser added into system, will get hello file in their home directory. 

```
- Managing users groups 
```bash 

groupadd sales          # add sales group 
groupmod -U jane sales  # add jane into sales group 
id jane                 # check jane group permission 
lid -g sales            # check all users under sales group 
```

- Manage Password Settings 
 - /etc/login.defs   # basic password req 
 - Change password settings 
 ```bash 
 chage      # convinient command 
 passwd 
 ```

 # Lesson 10 - Securiing Files with Permissions 

 - change ownership 
    - chown user[:group] file  #set user-ownership 
    - chgrp group file         #set group-ownership 
    - mkdir -p /data/newfiles   # create file and dir under the root 
        ```bash
        root@x:/data# ls -l 
        total 0
        drwxr-xr-x. 1 root root 0 Jun 18 14:07 newfiles
        ```
    -chown lisa newfiles/ 
        ```bash 
        root@x:/data# chown lisa newfiles/
        root@x:/data# ls -l
        drwxr-xr-x. 1 lisa root 0 Jun 18 14:07 newfiles

        root@x:/data# chown lisa:students newfiles/
        root@x:/data# ls -l
        drwxr-xr-x. 1 lisa students 0 Jun 18 14:07 newfiles

        chown :students newfiles/  # change groups permission only 

        ```
- change permission 
    - chmod 750 myfile 
    - chmod +x myscript     # make it executable 
    - chmod g+w newfiles/   # add write persmission to group 
    - chmod o+w newfiles/   # add write permission to owner 
    - chmod o-w newfiles/   # remove write pemission from owner
    - #if user own folder, he/she can delete files or folder under his/her own directory 
    - set command "umask 077" in .bashrc, make user can run script in bash shell  
---