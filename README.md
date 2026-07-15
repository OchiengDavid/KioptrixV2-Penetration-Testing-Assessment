# Kioptrix Level 2 — Penetration Test Report

A full black-box penetration test of the Kioptrix Level 2 vulnerable VM, performed as part of an Information Security & Digital Forensics coursework engagement. Scope covered the complete attack chain: host discovery → service enumeration → web application exploitation → initial foothold → local privilege escalation to root.

>Academic lab (isolated host-only network, `192.168.56.0/24`). No production systems involved.

## Summary


Target: Kioptrix Level 2 (CentOS, Apache 2.0.52, MySQL, PHP) 
Outcome : Full compromise — `uid=0` root shell 
Entry point : SQL injection → command injection in a web admin console 
Escalation path : Outdated Linux kernel → public local root exploit 
My role :led web application exploitation and reverse shell handling |

## Attack Chain

**1. Reconnaissance**
Host discovery on the internal network with `arp-scan`, followed by an aggressive `nmap -sV -O` scan to enumerate open ports, service versions, and fingerprint the OS. Identified Apache 2.0.52, MySQL, and rpcbind on a CentOS host running a 2.6.x kernel.

**2. Vulnerability Research**
Cross-referenced the fingerprinted Apache version against `searchsploit` to scope known CVEs, then pivoted to testing the web application layer directly, which offered a faster path in.

**3. Authentication Bypass (SQL Injection)**
The "Remote System Administration Login" form failed to sanitize input. Submitting `admin'--` as the username short-circuited the SQL query logic and granted access without valid credentials.

**4. Command Injection**
Inside the admin console, a "Ping a Machine on the Network" feature passed user input directly into a shell command. Appending `;ls;id;whoami` after a valid IP confirmed arbitrary command execution as the `apache` user.

**5. Foothold — Reverse Shell**
Used the command injection point to trigger a bash reverse shell back to a Netcat listener, converting blind command execution into an interactive session.

**6. Privilege Escalation**
Fingerprinted the running kernel version from the shell, identified a matching public local-root exploit, staged it on the target via `wget` (served from the attacker box with `python -m http.server`), compiled it in place with `gcc`, and executed it to obtain a root shell (`uid=0`).

## Tools Used

`arp-scan` · `nmap` · `searchsploit` · Burp Suite (Repeater) · `curl` · `netcat` · `wget` · `gcc` · Kali Linux

## Remediation Recommendations

- **Input sanitization** — whitelist/validate all fields feeding into system commands (e.g. strict IP-format validation on the ping tool).
- **Parameterized queries** — replace raw SQL string concatenation in the login flow with prepared statements to eliminate the injection class entirely.
- **Patch management** — keep OS kernels current; this single control would have closed off the entire privilege escalation path.

## Skills Demonstrated

Recon & enumeration · manual SQL injection · manual command injection · exploit research (Exploit-DB) · reverse shell handling · Linux privilege escalation · exploit compilation (C/gcc) · technical report writing & remediation guidance

---
*Report by Group F — Information Security & Forensics coursework. This writeup is my own summary of a group lab exercise; see attribution note above for my specific contribution.*
