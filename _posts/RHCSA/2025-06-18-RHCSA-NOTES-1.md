---
title: "RHCSA NOTES 1"
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

```bash 
  - search / and n
  - man -a passwd
  - sudo mandb -> man -k user | grep
```
- nano myfile
- vim

```bash
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

## Lesson 6 Using Essential File Management Tools

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


## Lesson 8 - Using root privileges

- su -                  # Switch to root with login shell
- su -c 'command'       # Run one root command
- sudo command          # Run command as root
- sudo -i               # Open root shell
- usermod -aG wheel user#Add user into wheel permission to grant root access
- visudo                # Safely edit sudoers file
- sudo mkdir /etc/sudoers.d/         # Optional config directory
- sudo visudo -f /etc/sudoers.d/dev  # Create rule file for user 'dev'
- whoami                #Check user  
- id                    #check user and group permssion 
- groups username       # Check user’s groups

#command restrictions 

- edit /etc/sudoers.d/jane  , edit for jane only 
- jane ALL=(ALL) NOPASSWD: /bin/systemctl restart apache2 #jane can restart apache2 service 
- jane ALL=/usr/bin/passwd,!/usr/bin/passwd root #jane can reset password for all users but not for root users
- jane ALL=/user/sbin/useradd, /usr/sbin/usermod, /user/sbin/userdel, ALL=/usr/bin/passwd,!/usr/bin/passwd root

#check logs 
/var/log/secure 

#Check SSh services 

- sudo systemctl status sshd
- sudo systemctl restart sshd 
- deny ssh root login

```bash  
- "/etc/ssh/sshd_config" 
    PermitRootLogin no 
```


## Lesson 9 - Managing Users and Groups

- User properties are managed in ** /etc/passwd **
- User password keep under ** /etc/shadow **  

- Terminology
    - Name : name of the account
    - UID : unique identifier for users
    - GID - group ID
    - Home directory : personal files
    - shell : program that will be started after successful authenticated

- #Add and set password for user F
- sudo useradd jane 
- sudo passwd jane 
- sudo useradd -m -s /bin/bash jane  # -m create home directory, -s create default shell 
- sudo useradd lori -u 2000 -s /sbin/nologin  #create user with id 2k and no login access 

- sudo usermod -L jane      #lock accout 
- sudo usermod -U jane      #Unlock account 
- sudo usermod -e 2025-08-01  jane  # set expire 
- sudo userdel jane 
- sudo userdel -rf jane      #Remove home directory 
- sudo usermod -s /sbin/nologin jane  #disable shell access 

```bash 
- User default permissions /etc/login.defs 
    - PASS_MAX_DAYS 
    - PASS_MIN_DAYS
    - PASS WARN AGE 
    - HOME_MODE 
```

- #create startup folder in new users 
 - touch /etc/skel/hello     #when newuser added into system, will get hello file in their home directory. 

- Managing users groups 

- groupadd sales          # add sales group 
- groupmod -U jane sales  # add jane into sales group 
- id jane                 # check jane group permission 
- lid -g sales            # check all users under sales group 


- Manage Password Settings 
 - /etc/login.defs   # basic password req 
 - Change password settings 
 
 ```bash 
 chage      # convinient command 
 passwd 
 ```


 ## Lesson 10 - Securiing Files with Permissions 

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



## Lesson 11 - Managing network configuration 

 - NIC Naming
    -  em[1-N] for embedded NICs
    -  eno [nn] for embedded NICs
    - p<slot>p<port> for NICs on the PCI bus
    - "ip link show" and "ip addr"can get bios nic name and mac-address
    
 
```bash
        etherhtun@x:~$ ip link show
        1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
            link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        2: enp0s31f6: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN mode DEFAULT group default qlen 1000
            link/ether 48:2a:e3:7e:5d:fe brd ff:ff:ff:ff:ff:ff
            altname enx482ae37e5dfe
        3: wlp82s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DORMANT group default qlen 1000
            link/ether 9e:9f:8f:84:12:a9 brd ff:ff:ff:ff:ff:ff permaddr 6c:6a:77:ed:76:41
            altname wlx6c6a77ed7641
      -------------------------------------------
        etherhtun@x:~$ ip link show
        1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
            link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        2: enp0s31f6: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN mode DEFAULT group default qlen 1000
            link/ether 48:2a:e3:7e:5d:fe brd ff:ff:ff:ff:ff:ff
            altname enx482ae37e5dfe
        3: wlp82s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DORMANT group default qlen 1000
            link/ether 9e:9f:8f:84:12:a9 brd ff:ff:ff:ff:ff:ff permaddr 6c:6a:77:ed:76:41
            altname wlx6c6a77ed7641
```
- Name resolution
    - /etc/hostname     # hostname, local resolver 
    - /etc/resolv.conf  # dns server config 
    - hostname          # checkhostname 
    - hostnamectl       # more detail
    - 

- Net config 
    - ip a 
    - ip -s link        #detail TX RX  and errors 
    - ip route show     # get routing table
    - ip -6 route 
    - tracepath , tracepath6  
    - ip route add 2.2.2.2/32 via 10.10.10.1   # adding persistant route 
    - /etc/NetworkManager/system-connections   # network manager location 
    - nmcli                                    # show netowork config
    - nmcli general permission                 # check user access permission
    - nmcli con show                           # show current connections
    - nmcli dev status                         # show device status  
    - nmtui                                    # edit interface with instruction, graphphical mode 
    - systemctl status NetworkManager          # check network manager status
    - "nmcli connection add  con-name mycon ifname wlp82s0 ipv4.addresses 192.168.51.51/24 ipv4.gateway 192.168.51.1 ipv4.method manual type wifi"
    
    - ss -tu                                   #  connection status, tu mean tcp/udp 


## Lesson 12 - Managing Software 

- RPM Packages 
    - rpm -qa                                   # Show the packages that currently installed 
    - rpm -qf filename                          # show from which package filename was installed
    - rpm -ql                                   # list files installed from a package
    - rpm -q --scripts                          # show scrpits executed while installation th packages 


- Understanding repo 
    - dnf list                                  # check available package to install 
    - dnf serch                                 # name and summary 
    - dnf search all seinfo                     # name, summary and description 
    - dnf install                               # install software 
    - dnf remove                                # remove software
    - dnf update                                # update software 
     


- Manual Setup Repo, (Option 1 , copy iso to / and mount to repo) 
    1) Create Local repo server 

    - df -h                                     # Check the diskspace 
    - lsblk                                     # Check the disk directory 
    - dd if=/dev/sr0 of=/rhel9.iso bs=1M        # low level copy file from sr0 to /rhel9.iso with blocksize 1M 
    - mkdir /repo                               # create /repo directory to mount rhel9.iso
    - echo "/rhel9.iso /repo iso9660 defaults 0 0" >> /etc/fstab   # create mountpioint to /repo from /rhel9.iso using iso images (9660) no check, no dump) 

    2) Update repo list 
    
```bash
    [root@localhost repo]# cat /etc/yum.repos.d/rhel9.repo 
    [BaseOS]
    name=BaseOS Packages Red Hat Enterprise Linux 9
    metadata_expire=-1
    gpgcheck=0
    enabled=1
    baseurl=file:///repo/BaseOS/

    [AppStream]
    name=AppStream Packages Red Hat Enterprise Linux 9
    metadata_expire=-1
    gpgcheck=0
    enabled=1
``` 

- dnf repolist 
- dnf search all namp 
- dnf insall nmap 
- dnf remove nmap 
- dnf history
- dnf history undo 3 

[outputs](/Documents/RHCSA/Lesson12-Managing-Software.txt)   
     
---