# ✈️ Airplane — TryHackMe CTF Walkthrough

<p align="center">
  <img src="docs/assets/banner.png" alt="Airplane TryHackMe Banner" width="100%">
</p>

<p align="center">
  <a href="https://tryhackme.com/">
    <img src="https://img.shields.io/badge/TryHackMe-Airplane-red?style=for-the-badge&logo=tryhackme" />
  </a>
  <img src="https://img.shields.io/badge/Difficulty-Medium-orange?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Platform-Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black" />
  <img src="https://img.shields.io/badge/Focus-Penetration%20Testing-blue?style=for-the-badge" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/LFI-Exploitation-6f42c1?style=flat-square"/>
  <img src="https://img.shields.io/badge/gdbserver-RCE-critical?style=flat-square"/>
  <img src="https://img.shields.io/badge/SUID-Privilege%20Escalation-success?style=flat-square"/>
  <img src="https://img.shields.io/badge/Documentation-Portfolio%20Project-0A66C2?style=flat-square"/>
</p>

---

## 📌 Overview

**Airplane** is a Linux-based Capture The Flag (CTF) room available on **TryHackMe** that simulates a realistic penetration testing engagement against a vulnerable web server.

This repository documents the **complete exploitation lifecycle**, beginning with reconnaissance and service enumeration, progressing through **Local File Inclusion (LFI)** and **Remote Code Execution (RCE)**, and ending with multiple privilege escalation techniques leading to **root access**.

Unlike a traditional write-up, this project is structured as a **professional penetration testing report** intended for cybersecurity learning, portfolio presentation, and documentation best practices.

> **Purpose:** Demonstrate practical offensive security methodology, Linux enumeration, privilege escalation, documentation, and reporting skills within an authorized TryHackMe laboratory.

---

# 🎯 Objectives

This walkthrough demonstrates how to:

* Perform Linux reconnaissance using Nmap.
* Enumerate HTTP services and identify hidden attack surfaces.
* Exploit a Local File Inclusion vulnerability.
* Enumerate Linux processes through the `/proc` filesystem.
* Discover and exploit an exposed `gdbserver` instance.
* Obtain an initial reverse shell.
* Escalate privileges using SUID binaries.
* Gain persistent authenticated access using SSH keys.
* Exploit a vulnerable `sudo` wildcard configuration.
* Document findings using a professional penetration testing methodology.

---

# 🧠 Skills Demonstrated

| Domain                   | Techniques                                                         |
| ------------------------ | ------------------------------------------------------------------ |
| **Reconnaissance**       | Nmap TCP SYN Scan, Version Detection, NSE Scripts                  |
| **Web Enumeration**      | Directory Traversal, Local File Inclusion                          |
| **Linux Enumeration**    | `/etc/passwd`, `/etc/group`, `/proc/self/environ`, `/proc/net/tcp` |
| **Process Discovery**    | PID Enumeration, `cmdline` Analysis                                |
| **Exploitation**         | gdbserver Remote Code Execution                                    |
| **Initial Access**       | Reverse TCP Shell                                                  |
| **Privilege Escalation** | SUID `find`, SSH Persistence, Wildcard `sudo` Misconfiguration     |
| **Reporting**            | MITRE ATT&CK Mapping, Security Findings, Remediation               |

---

# ⚙️ Lab Information

| Property         | Value                                   |
| ---------------- | --------------------------------------- |
| Platform         | TryHackMe                               |
| Room             | Airplane                                |
| Operating System | Linux                                   |
| Difficulty       | Medium                                  |
| Category         | Web Exploitation / Privilege Escalation |
| Environment      | Authorized Lab                          |

---

# 🛠️ Tools Used

| Tool             | Purpose                                        |
| ---------------- | ---------------------------------------------- |
| **Nmap**         | Network reconnaissance and service enumeration |
| **Burp Suite**   | HTTP interception and request manipulation     |
| **Searchsploit** | Exploit discovery                              |
| **Python**       | Exploit execution and enumeration              |
| **msfvenom**     | Reverse shell payload generation               |
| **Netcat**       | Listener for reverse shell                     |
| **GTFOBins**     | Privilege escalation research                  |
| **SSH**          | Persistent authenticated shell                 |

---

# 🔍 Attack Methodology

The assessment followed a structured penetration testing workflow inspired by the **PTES (Penetration Testing Execution Standard)** methodology.

```text
Reconnaissance
      │
      ▼
Service Enumeration
      │
      ▼
Web Application Analysis
      │
      ▼
Local File Inclusion (LFI)
      │
      ▼
Linux Enumeration (/proc)
      │
      ▼
Hidden Service Discovery
      │
      ▼
gdbserver Exploitation
      │
      ▼
Reverse Shell (hudson)
      │
      ▼
Privilege Escalation
      │
      ├── SUID Binary Abuse
      ├── SSH Key Persistence
      └── Wildcard sudo Misconfiguration
      │
      ▼
ROOT ACCESS
```

Every stage is documented with:

* Objective
* Commands executed
* Observations
* Security impact
* Screenshots
* Technical explanation

---

# 🗂️ Repository Structure

```text
TryHackMe-Airplane-CTF-Walkthrough/
│
├── README.md
├── _config.yml
│
├── Documentation/
│   ├── THM_Airplane_Documentation.md
│   ├── THM_Airplane_Report.pdf
│
├── Resources/
│   ├── notes.md
│   ├── payloads.md
│   ├── tools.md
│   ├── references.md
│   └── remediation.md
│
├── docs/
│   ├── index.md
│   └── assets/
│       ├── banner.png
│       ├── css/
│         ├── custom.scss
│
└── Screenshots/
    ├── figure-1-hosts-file.png
    ├── figure-2-nmap-scan.png
    ├── figure-3-service-scan.png
    ├── figure-4-lfi-discovery.png
    ├── figure-5-passwd-enumeration.png
    ├── figure-6-proc-enumeration.png
    ├── figure-7-gdbserver.png
    ├── figure-8-reverse-shell.png
    ├── figure-9-suid-find.png
    ├── figure-10-ssh-persistence.png
    ├── figure-11-sudo-ruby.png
    └── figure-12-root-shell.png
```

---

# 🕵️ Attack Surface Summary

| Attack Surface | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| **Port 22**    | SSH service for authenticated access.                        |
| **Port 8000**  | Python web application exposing a vulnerable file parameter. |
| **Port 6048**  | Hidden debugger service later identified as `gdbserver`.     |

The walkthrough demonstrates how seemingly unrelated services combine into a complete compromise chain.

---

# 💥 Exploitation Highlights

## Phase 1 — Reconnaissance

* Host discovery.
* TCP SYN scan.
* Version enumeration.
* Service fingerprinting.

**Outcome**

Identified multiple services including a hidden service not recognized by default Nmap signatures.

---

## Phase 2 — Local File Inclusion

* Tested URL parameters for path traversal.
* Retrieved Linux system files.
* Enumerated local users.
* Collected privilege information.

**Outcome**

Confirmed a Local File Inclusion vulnerability that enabled unrestricted file disclosure.

---

## Phase 3 — Linux Enumeration

Leveraged the `/proc` filesystem to enumerate:

* Running user.
* Network sockets.
* Active processes.
* Process command lines.

**Outcome**

Discovered an exposed debugger service running internally.

---

## Phase 4 — Remote Code Execution

* Identified exposed `gdbserver`.
* Generated reverse shell payload.
* Executed public RCE exploit.
* Established shell access.

**Outcome**

Obtained an interactive shell on the target machine.

---

## Phase 5 — Privilege Escalation

Documented multiple privilege escalation techniques including:

* SUID Binary Enumeration.
* GTFOBins Research.
* SSH Key Persistence.
* Wildcard `sudo` Path Traversal.

**Outcome**

Successfully achieved root privileges through misconfigured permissions.

---

# 🛡️ Security Findings

| Finding                         | Severity    |
| ------------------------------- | ----------- |
| Local File Inclusion            | 🔴 High     |
| Exposed gdbserver Service       | 🔴 Critical |
| SUID Binary Abuse               | 🟠 High     |
| SSH Authorized Keys Persistence | 🟡 Medium   |
| Wildcard sudo Misconfiguration  | 🔴 Critical |

Each finding includes remediation recommendations inside the documentation.

---

# 🧬 MITRE ATT&CK Mapping

| Tactic               | Technique                                 |
| -------------------- | ----------------------------------------- |
| Initial Access       | Exploit Public-Facing Application (T1190) |
| Discovery            | Process Discovery (T1057)                 |
| Discovery            | System Information Discovery (T1082)      |
| Execution            | Command and Scripting Interpreter (T1059) |
| Persistence          | SSH Authorized Keys (T1098.004)           |
| Privilege Escalation | Abuse Elevation Control Mechanism (T1548) |

This mapping connects offensive actions with defensive frameworks commonly used in SOC environments.

---

# 📖 Documentation

Complete technical documentation is available in the repository.

| Document                          | Description                                             |
| --------------------------------- | ------------------------------------------------------- |
| **THM_Airplane_Documentation.md** | Complete walkthrough with explanations and screenshots. |
| **THM_Airplane_Report.docx**      | Professional penetration testing report.                |
| **docs/index.md**                 | GitHub Pages documentation website.                     |
| **Resources/**                    | Payloads, tools, remediation notes, and references.     |

---

# 🖼️ Evidence

Screenshots are intentionally stored separately under the **Screenshots/** directory.

Each figure is referenced throughout the documentation using professional captions rather than embedding large images into the README.

| Figure    | Description                    |
| --------- | ------------------------------ |
| Figure 1  | Host Resolution Configuration  |
| Figure 2  | Full Port Scan                 |
| Figure 3  | Service Enumeration            |
| Figure 4  | Local File Inclusion Discovery |
| Figure 5  | `/etc/passwd` Enumeration      |
| Figure 6  | `/proc` Enumeration            |
| Figure 7  | gdbserver Identification       |
| Figure 8  | Reverse Shell                  |
| Figure 9  | SUID Enumeration               |
| Figure 10 | SSH Key Persistence            |
| Figure 11 | Wildcard sudo Exploit          |
| Figure 12 | Root Shell Verification        |

---

# 🚩 Flag Policy

To preserve the educational integrity of the TryHackMe room and discourage plagiarism, the actual challenge flags are **intentionally redacted**.

```text
user.txt  → THM{********************************}

root.txt  → THM{********************************}
```

The repository focuses on the **methodology and exploitation process**, not publishing challenge answers.

---

# 📚 Key Learning Outcomes

After completing this room, the following practical skills were reinforced:

* Web vulnerability assessment.
* Linux privilege enumeration.
* `/proc` filesystem analysis.
* Debugger service exploitation.
* Reverse shell stabilization.
* Linux privilege escalation methodology.
* SSH persistence techniques.
* Security reporting and remediation documentation.

---

# 🔐 Remediation Summary

| Vulnerability         | Recommended Mitigation                                                   |
| --------------------- | ------------------------------------------------------------------------ |
| Local File Inclusion  | Validate user-controlled file paths and implement allowlists.            |
| gdbserver Exposure    | Disable remote debugging or restrict access via firewall/authentication. |
| SUID Misconfiguration | Remove unnecessary SUID permissions from binaries.                       |
| SSH Persistence Risk  | Restrict write permissions to `.ssh/authorized_keys`.                    |
| Wildcard sudo Rules   | Avoid wildcard paths inside `sudoers` and use explicit file permissions. |

---

# 🌐 GitHub Pages

A documentation website is included using **GitHub Pages**.

The website contains:

* Executive Summary
* Methodology
* Attack Chain
* Privilege Escalation Analysis
* Security Findings
* References

Enable Pages by deploying the `/docs` directory from the `main` branch.

---

# ⚠️ Disclaimer

This repository documents exploitation techniques performed **only inside an authorized TryHackMe laboratory**.

The information is provided exclusively for:

* Cybersecurity education.
* Defensive security learning.
* Capture The Flag documentation.
* Authorized penetration testing practice.

Do **not** use these techniques against systems without explicit authorization.

---

# 👨‍💻 Author

## **Anurag Ravankar**

Cybersecurity Enthusiast • Penetration Testing • SOC • Linux Security

* Linux Security
* Web Application Security
* Capture The Flag (CTF) Writeups
* TryHackMe Documentation
* Security Research

---

<p align="center">
  ⭐ If this documentation helps you understand Linux privilege escalation or CTF reporting methodology, consider starring the repository.
</p>
