# TryHackMe: Bolt - Writeup & Penetration Testing Walkthrough

## 📝 Executive Summary
This repository contains a comprehensive, methodology-driven penetration testing writeup for the **Bolt** laboratory on TryHackMe. The objective of this assessment was to identify security vulnerabilities, achieve initial access, and perform privilege escalation to gain full administrative control (`root`) over the target system.

The assessment followed the **Penetration Testing Execution Standard (PTES)** and mapped techniques to the **MITRE ATT&CK®** framework.

### 🛡️ Key Findings & Exploded Vulnerabilities
1. **Information Disclosure via NFS Share:** An overly permissive NFS export allowed unauthorized remote users to mount internal directories and extract sensitive backups (`save.zip`).
2. **Local File Inclusion (LFI) / Directory Traversal:** The BoltWire CMS instance on port 8080 was vulnerable to arbitrary file read via input sanitization failure (`../../../../etc/passwd`).
3. **Weak Sudo Configuration (Privilege Escalation):** The user account possessed excessive sudo privileges, allowing the execution of `/usr/bin/zip` as root with `NOPASSWD`, violating the Principle of Least Privilege.

---

## 🗺️ Attack Lifecycle & Repository Structure

The writeup is systematically documented across the following phases:

### 🔍 [Phase 1: Reconnaissance](./01-Reconnaissance/active-recon.md)
* **Objective:** Host discovery and open port identification.
* **Tools:** `arp-scan`, `netdiscover`, `nmap`.
* **Key Outcome:** Identification of open services on ports 22 (SSH), 80/8080 (HTTP), and 2049 (NFS). Passive OS fingerprinting via TTL analysis.

### 📂 [Phase 2: Enumeration](./02-Enumeration/web-nfs-services.md)
* **Objective:** Surface mapping and vulnerability scanning.
* **Tools:** `whatweb`, `gobuster`, `showmount`.
* **Key Outcome:** Extraction of an encrypted backup file (`save.zip`) from the public NFS share and directory brute-forcing on web endpoints.

### ⚡ [Phase 3: Exploitation](./03-Exploitation/initial-access.md)
* **Objective:** Initial access and credential harvesting.
* **Tools:** `fcrackzip`, `crackmapexec`, `hydra`, `msfconsole`.
* **Key Outcome:** Cracking the ZIP archive using `rockyou.txt`, harvesting SSH private keys (`id_rsa`), and exploiting Bolt CMS authentication mechanisms.

### 👑 [Phase 4: Post-Exploitation](./04-Post-Exploitation/privilege-escalation.md)
* **Objective:** Local enumeration and Privilege Escalation.
* **Tools:** `linpeas.sh`, `sudo`, `zip`.
* **Key Outcome:** System exploitation through a `sudo` misconfiguration on the `zip` binary, abusing its execution parameters via **GTFOBins** to achieve an interactive `root` shell.

---

## 🛠️ Cyber Security Best Practices Demonstrated
* **Structured Documentation:** Separating technical evidence into logical phases as done in professional industrial assessments.
* **Proactive Remediation:** Every phase includes a mitigation section detailing how to fix the vulnerability from a defensive standpoint.
* **Safe Credential Management:** Implementation of secure private key handling (adjusting overly permissive 0744 file permissions to strict 0600 standards).

---
*Disclaimer: This repository is intended strictly for educational purposes and professional portfolio demonstration. All tests were conducted within a controlled legal laboratory environment (TryHackMe).*
