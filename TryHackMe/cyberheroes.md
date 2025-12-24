# 🦸‍♂️ CyberHeroes CTF Write-up 🏴‍☠️

## 📑 Table of Contents
- Overview
- Reconnaissance
  - Network Scanning
  - Website Exploration
- Vulnerability Discovery
  - Login Page Analysis
  - JavaScript Inspection
- Exploitation
  - Password Decoding
  - Flag Capture
- Key Takeaways
- Conclusion

---

## 📋 Overview

This write-up documents my solution to the CyberHeroes CTF challenge on TryHackMe.
The challenge focused on web application security with the simple objective: "Prove your merit by finding a way to log in!"

The investigation revealed a classic client-side authentication vulnerability where login credentials were hidden in plain sight within JavaScript code.

---

## 🔍 Reconnaissance
### 🌐 Network Scanning

I began with an Nmap scan to map the target's attack surface and identify open services.

```bash
nmap -sV -sC 10.48.160.187
```
Scan Results:
```bash
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-24 05:56 PST
Nmap scan report for 10.48.160.187
Host is up (0.12s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 a3:80:d6:31:53:50:f7:86:27:53:42:09:e4:8c:2e:99 (RSA)
|   256 77:5a:b7:f8:3f:1e:fe:ac:a5:45:50:55:57:7f:74:31 (ECDSA)
|_  256 47:74:2f:73:c7:24:b7:e0:ee:85:2c:83:cd:46:c9:28 (ED25519)
80/tcp open  http    Apache httpd 2.4.48 ((Ubuntu))
|_http-title: CyberHeros : Index
|_http-server-header: Apache/2.4.48 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Key Findings:
 - 🌐 Port 80 (HTTP): Web server running Apache 2.4.48
 - 🔐 Port 22 (SSH): OpenSSH service (not needed for this challenge)
 - 🐧 Operating System: Ubuntu Linux

### 🖥️ Website Exploration

Navigating to http"//10.48.160.187 revealed a landing page with a homepage, about and a login page.
Screenshot CyberHeroes homepage with login

---

## 🕵️ Vulnerability Discovery
### 🔓 Login Page Analysis

The login form appeared to perform client-side validation. Instead of attempting brute force attacks, I examined the client-side JavaScript handling the authentication.

### 📜 JavaScript Inspection
Opening the browser's Developer Tools (F12) and viewing the Page Source revealed the authentication logic embedded directly in the HTML:

```bash
javascript
<script>
    function authenticate() {
      a = document.getElementById('uname')
      b = document.getElementById('pass')
      const RevereString = str => [...str].reverse().join('');
      if (a.value=="h3ck3rBoi" & b.value==RevereString("54321@terceSrepuS")) { 
        var xhttp = new XMLHttpRequest();
        xhttp.onreadystatechange = function() {
          if (this.readyState == 4 && this.status == 200) {
            document.getElementById("flag").innerHTML = this.responseText ;
            document.getElementById("todel").innerHTML = "";
            document.getElementById("rm").remove() ;
          }
        };
        xhttp.open("GET", "RandomLo0o0o0o0o0o0o0o0o0o0gpath12345_Flag_"+a.value+"_"+b.value+".txt", true);
        xhttp.send();
      }
      else {
        alert("Incorrect Password, try again.. you got this hacker !")
      }
    }
  </script>
```

Critical Findings:

 - 👤 Username
 - 🔄 Password Logic: The password is compared to the reversed string "54321@terceSrepuS"

🚨 Vulnerability: All authentication logic is performed client-side, making it trivial to bypass

Screenshot placeholder: Browser Developer Tools showing the JavaScript code

---

## ⚔️ Exploitation
### 🔄 Password Decoding

The password validation checks if the input equals RevereString("54321@terceSrepuS"). The function RevereString simply reverses a string.
I used the Linux rev command to reverse the string and obtain the password:

```bash
echo "54321@terceSrepuS" | rev
```

Credentials Obtained:

 - 👤 Username: h3ck3rBoi
 - 🔑 Password: SuperSecret######

### 🚩 Flag Capture

With the credentials discovered, I logged into the application:

1. Navigated to the login page
2. Entered username & password
3. Clicked the login button

Authentication Successful 

🏴‍☠️ Flag displayed on the page

Screenshot placeholder: Successful login page showing the flag message

---

## 📊 Key Takeaways

 - 🔓 Username	h3ck3rBoi
 - 🔑 Password	Reversed String
 - 🚨 Vulnerability	Client-Side Authentication	Login Logic	Critical Security Flaw
 - 🏁 Flag	with Successful Login,	Challenge Complete

Security Lessons Learned:

 - ⚠️ Never trust client-side validation for authentication
 - 🔒 Server-side checks are essential for security
 - 👁️ Security through obscurity (like string reversal) provides no real protection
 - 📜 Always inspect client-side code during penetration tests

---

## ✅ Conclusion
The CyberHeroes challenge provided a clear demonstration of a common web application vulnerability—client-side authentication. By methodically examining the available source code, I bypassed the weak security controls and obtained the flag.

## 🎯 CTF Completed!
