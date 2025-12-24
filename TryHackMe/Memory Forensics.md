# 🧠 Memory Forensics CTF Write-up 🏴‍☠️

## 📑 Table of Contents
- Overview
- Setup & Preparation
  - Volatility Installation
- Task 1: Credential Extraction
  - Memory Profile Identification
  - Hash Dumping
  - Password Cracking
- Task 2: Timeline Analysis
  - System Shutdown Time
  - Command History Recovery
- Task 3: TrueCrypt Investigation
  - Passphrase Extraction
- Key Findings Summary
- Conclusion

---

## 📋 Overview

This write-up documents my analysis of a **Memory Forensics CTF challenge** involving multiple Windows memory dumps.  
The investigation focused on:

- 🔐 Extracting credentials
- ⏰ Analyzing user activity and timelines
- 🗝️ Recovering encryption keys from volatile memory

The analysis was performed across **three distinct memory snapshots**.

---

## 🛠️ Setup & Preparation
### 📥 Volatility Installation

Volatility **v2.6** was installed on a Linux system to begin memory analysis.

```bash
# Download Volatility 2.6
wget http://downloads.volatilityfoundation.org/releases/2.6/volatility_2.6_lin64_standalone.zip

# Extract the archive
unzip volatility_2.6_lin64_standalone.zip

# Install to system path
sudo cp volatility_2.6_lin64_standalone/volatility_2.6_lin64_standalone /usr/local/bin/volatility
sudo chmod +xr /usr/local/bin/volatility
```
---

## 🔍 Task 1: Credential Extraction
### 🖥️ Memory Profile Identification

The operating system profile was identified using the imageinfo plugin.

```bash
volatility imageinfo -f Snapshot6_1609157562389.vmem 
Volatility Foundation Volatility Framework 2.6
INFO    : volatility.debug    : Determining profile based on KDBG search...
          Suggested Profile(s) : Win7SP1x64, Win7SP0x64, Win2008R2SP0x64, Win2008R2SP1x64_23418, Win2008R2SP1x64, Win7SP1x64_23418
                     AS Layer1 : WindowsAMD64PagedMemory (Kernel AS)
                     AS Layer2 : FileAddressSpace (/home/xtony/Desktop/CTF/Snapshot6_1609157562389.vmem)
                      PAE type : No PAE
                           DTB : 0x187000L
                          KDBG : 0xf80002c4a0a0L
          Number of Processors : 1
     Image Type (Service Pack) : 1
                KPCR for CPU 0 : 0xfffff80002c4bd00L
             KUSER_SHARED_DATA : 0xfffff78000000000L
           Image date and time : 2020-12-27 06:20:05 UTC+0000
     Image local date and time : 2020-12-26 22:20:05 -0800
```

### 🔓 Hash Dumping

Password hashes were extracted.

```bash
sudo volatility -f Snapshot6_1609157562389.vmem --profile Win7SP1x64 hashdump --output-file=snapshot.creds  
Volatility Foundation Volatility Framework 2.6
Outputting to: snapshot.creds
```
```bash
cat snapshot.creds 
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
John:1001:aad3b435b51404eeaad3b435b51404ee:47fbd6536d7868c873d5ea455f2fc0c9:::
HomeGroupUser$:1002:aad3b435b51404eeaad3b435b51404ee:91c34c06b7988e216c3bfeb9530cabfb:::
```

### 🔨 Password Cracking

John the Ripper was used with the rockyou wordlist.

```bash
john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt snapshot.creds
```

Cracking Results:
- ✅ John's password
- ⚠️ Administrator: Blank

---

## ⏰ Task 2: Timeline Analysis
### 📅 System Shutdown Time

The shutdown time was extracted from registry artifacts.

```bash
#Identify memory profile
volatility imageinfo -f Snapshot19_1609159453792.vmem
```
```bash
### Check shutdown time
sudo volatility -f Snapshot19_1609159453792.vmem shutdowntime --profile Win7SP1x64
```

### 💻 Command History Recovery

User command history was reconstructed using the consoles plugin.

```bash
sudo volatility -f Snapshot19_1609159453792.vmem consoles --profile Win7SP1x64
```

Discovery:
- 🚩 Hidden Flag
- 📁 Directory exploration activity
- ⚠️ Incorrect command syntax attempts

---

## 🔐 Task 3: TrueCrypt Investigation
### 🗝️ Passphrase Extraction

TrueCrypt passphrases were recovered directly from memory.

```bash
# Identify memory profile
sudo volatility -f Snapshot14_1609164553061.vmem imageinfo
```
```bash
# Extract TrueCrypt passphrase
sudo volatility -f Snapshot14_1609164553061.vmem truecryptpassphrase --profile Win7SP1x64
```

Discovery:  
- 🗝️ Found TrueCrypt passphrase

## 📊 Key Findings Summary

- Finding	Value	Source	Impact
- John's Password	Snapshot6
- Administrator Password	Blank	Snapshot6
- Last Shutdown	Snapshot19
- Hidden Flag	Snapshot19
- TrueCrypt Passphrase	Snapshot14

## ✅ Conclusion
This Memory Forensics CTF provided hands-on experience in analyzing Windows memory dumps for critical forensic artifacts.

## 🎯 CTF Completed!
