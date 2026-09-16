---

layout: default
title: "Airplane — TryHackMe CTF"
description: "Professional technical walkthrough and penetration testing documentation for the TryHackMe Airplane room."
------------------------------------------------------------------------------------------------------------------------

<div align="center">

# ✈️ Airplane — TryHackMe

### Professional CTF Walkthrough & Penetration Testing Documentation

<p>
  <a href="https://tryhackme.com/">
    <img src="https://img.shields.io/badge/TryHackMe-Airplane-e11d48?style=for-the-badge&logo=tryhackme&logoColor=white" alt="TryHackMe Airplane">
  </a>
  <img src="https://img.shields.io/badge/Platform-Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black" alt="Linux">
  <img src="https://img.shields.io/badge/Focus-Penetration%20Testing-2563eb?style=for-the-badge" alt="Penetration Testing">
  <img src="https://img.shields.io/badge/Flags-Redacted-64748b?style=for-the-badge" alt="Flags Redacted">
</p>

<p>
  <strong>Reconnaissance → LFI → /proc Enumeration → gdbserver RCE → Shell → SUID → SSH → sudo → Root</strong>
</p>

</div>

---

## 📌 About This Project

This site presents the technical documentation for my completion of the **TryHackMe Airplane** Linux CTF.

The assessment was approached as a structured penetration-testing exercise rather than as a collection of isolated commands. The objective was to identify the attack surface, validate weaknesses, establish an initial foothold, enumerate the host, escalate privileges, and document the complete attack path.

> **Lab Scope**
>
> All testing documented here was performed against an authorized TryHackMe training environment.

---

## 🧭 Attack Chain

```text
┌───────────────────────┐
│   Reconnaissance      │
│   Nmap Enumeration    │
└──────────┬────────────┘
           │
           ▼
┌───────────────────────┐
│   Web Enumeration     │
│   Port 8000           │
└──────────┬────────────┘
           │
           ▼
┌───────────────────────┐
│ Local File Inclusion  │
│      LFI / Traversal  │
└──────────┬────────────┘
           │
           ▼
┌───────────────────────┐
│   Linux /proc Enum    │
│   TCP + Process Data  │
└──────────┬────────────┘
           │
           ▼
┌───────────────────────┐
│ Hidden Service        │
│    gdbserver:6048    │
└──────────┬────────────┘
           │
           ▼
┌───────────────────────┐
│ Remote Code Execution │
│   Reverse Shell       │
└──────────┬────────────┘
           │
           ▼
┌───────────────────────┐
│ Privilege Escalation  │
│   SUID `find`         │
└──────────┬────────────┘
           │
           ▼
┌───────────────────────┐
│ SSH Key Persistence   │
└──────────┬────────────┘
           │
           ▼
┌───────────────────────┐
│ sudo Wildcard Abuse   │
└──────────┬────────────┘
           │
           ▼
       ┌───────┐
       │ ROOT  │
       └───────┘
```

---

# 🔬 Assessment Snapshot

| Category                 | Details                               |
| ------------------------ | ------------------------------------- |
| **Platform**             | TryHackMe                             |
| **Room**                 | Airplane                              |
| **Operating System**     | Linux                                 |
| **Difficulty**           | Medium                                |
| **Assessment Style**     | Black-box lab assessment              |
| **Primary Web Issue**    | Local File Inclusion                  |
| **Hidden Service**       | gdbserver                             |
| **Initial Access**       | Remote code execution / reverse shell |
| **Privilege Escalation** | SUID binary + sudo misconfiguration   |
| **Final Objective**      | Root-level access                     |

---

# 🧠 What This CTF Demonstrates

### Reconnaissance

Identifying exposed services and distinguishing known services from unidentified network listeners.

### Web Security

Testing user-controlled file parameters and validating local file disclosure through path traversal.

### Linux Enumeration

Using the `/proc` filesystem to correlate runtime process information with network listeners.

### Exploitation

Turning an exposed debugger service into an initial foothold within the lab.

### Privilege Escalation

Investigating SUID binaries and later abusing an overly broad `sudo` rule.

### Professional Reporting

Recording evidence, observations, impact, remediation, and the reasoning behind each pivot.

---

# 📸 Evidence-Driven Documentation

The walkthrough is supported with **13 process screenshots**, each placed at the point where the corresponding action occurs.

| Figure | Evidence                         |
| ------ | -------------------------------- |
| **01** | Host resolution configuration    |
| **02** | Full TCP Nmap scan               |
| **03** | Service/version enumeration      |
| **04** | LFI discovery                    |
| **05** | `/etc/passwd` enumeration        |
| **06** | `/proc/net/tcp` enumeration      |
| **07** | `gdbserver` process discovery    |
| **08** | Successful reverse shell         |
| **09** | SUID enumeration                 |
| **10** | SUID `find` privilege escalation |
| **11** | SSH key authentication           |
| **12** | Wildcard `sudo` exploitation     |
| **13** | Root privilege verification      |

---

# 🧪 Methodology

The assessment was performed using a repeatable workflow:

```text
01  Reconnaissance
02  Enumeration
03  Vulnerability Validation
04  Exploitation
05  Initial Access
06  Post-Exploitation
07  Privilege Escalation
08  Verification
09  Documentation
10  Remediation
```

Each stage is connected to the next through evidence gathered during the assessment.

---

# 🛠️ Toolset

| Tool             | Role                                    |
| ---------------- | --------------------------------------- |
| **Nmap**         | Port and service discovery              |
| **Burp Suite**   | HTTP interception and parameter testing |
| **Python**       | Enumeration and exploit execution       |
| **Searchsploit** | Public exploit research                 |
| **msfvenom**     | Payload generation                      |
| **Netcat**       | Reverse-shell listener                  |
| **GTFOBins**     | SUID privilege-escalation research      |
| **SSH**          | Stable authenticated access             |

---

# 🧩 Key Technical Findings

## Local File Inclusion

The web application accepted a file-related parameter that could be manipulated to access local filesystem resources.

**Impact:** Information disclosure and improved attacker visibility into the host.

---

## Exposed gdbserver

Runtime enumeration revealed a debugger service listening on port `6048`.

**Impact:** The exposed debugging interface provided an avenue for remote code execution in the lab.

---

## SUID `find`

Local privilege enumeration identified an SUID-enabled `find` binary.

**Impact:** The binary could be leveraged for elevated command execution.

---

## Wildcard `sudo` Rule

A later `sudo` configuration permitted execution through a wildcard path.

**Impact:** Path manipulation allowed the restriction to be bypassed and root-level execution obtained.

---

# 🛡️ Defensive Perspective

The same attack path can be viewed from a blue-team perspective:

| Offensive Observation | Defensive Control                         |
| --------------------- | ----------------------------------------- |
| LFI file disclosure   | Input validation / allowlists             |
| Unknown listener      | Service inventory / network monitoring    |
| Exposed debugger      | Firewall restrictions / disable debugging |
| Reverse shell         | Egress monitoring / process telemetry     |
| SUID abuse            | SUID auditing / file integrity monitoring |
| SSH key persistence   | Authorized-key monitoring                 |
| sudo wildcard abuse   | Least-privilege sudo policies             |

---

# 🧬 MITRE ATT&CK Context

The assessment contains behaviors that can be mapped to common ATT&CK techniques, including:

| Tactic               | Technique                                     |
| -------------------- | --------------------------------------------- |
| Initial Access       | **T1190 — Exploit Public-Facing Application** |
| Discovery            | **T1057 — Process Discovery**                 |
| Discovery            | **T1046 — Network Service Discovery**         |
| Execution            | **T1059 — Command and Scripting Interpreter** |
| Persistence          | **T1098.004 — SSH Authorized Keys**           |
| Privilege Escalation | **T1548 — Abuse Elevation Control Mechanism** |

---

# 🚩 Flag Policy

The actual TryHackMe challenge flags are intentionally **not published** on this site.

```text
user.txt  → THM{********************************}

root.txt  → THM{********************************}
```

The purpose of this project is to demonstrate the **technical process, investigation workflow, exploitation methodology, and security reasoning** without distributing challenge answers.

---

# ✅ Completion Evidence

The documented assessment reached each major objective:

* ✅ Network reconnaissance completed
* ✅ Services enumerated
* ✅ LFI vulnerability identified
* ✅ Linux runtime information obtained
* ✅ Hidden debugger service identified
* ✅ Initial shell obtained
* ✅ SUID privilege escalation completed
* ✅ Stable SSH access established
* ✅ sudo misconfiguration exploited
* ✅ Root privileges verified
* ✅ Evidence documented

---

# 📖 Documentation

### Main Technical Walkthrough

**[Read the Complete Airplane Walkthrough](../Documentation%20/THM_Airplane_Documentation.md)**

The full report contains the detailed exploitation narrative, commands, observations, screenshots, security findings, remediation guidance, and final assessment outcome.

### Repository Resources

* **[Technical Notes](../Resources%20/notes.md)**
* **[Payload Reference](../Resources%20/payloads.md)**
* **[Tool Reference](../Resources%20/tools.md)**
* **[References](../Resources%20/references.md)**
* **[Remediation Guide](../Resources%20/remediation.md)**

---

# 📂 Evidence Repository

All process screenshots are maintained separately to keep the documentation clean and easy to navigate.

**[Open Screenshot Evidence →](../Screenshots/)**

---

# 💡 Key Lessons

### 01 — Enumeration Drives Exploitation

The initial unidentified service became significant only after correlating network data with host-level information.

### 02 — Small Weaknesses Can Chain Together

LFI, information disclosure, exposed services, SUID permissions, and sudo configuration were not isolated problems; together they formed a complete attack path.

### 03 — Context Matters

Understanding why a command is executed is more valuable than memorizing commands. Each step in this assessment was driven by an observation from the previous phase.

### 04 — Evidence Is Part of the Assessment

Screenshots, observations, and remediation recommendations turn a CTF solution into a reproducible security report.

---

# 🏁 Final Assessment

```text
╔════════════════════════════════════════╗
║       AIRPLANE CTF — COMPLETED         ║
╠════════════════════════════════════════╣
║ Reconnaissance ................. ✅     ║
║ Web Enumeration ................ ✅     ║
║ LFI Discovery .................. ✅     ║
║ /proc Enumeration .............. ✅     ║
║ gdbserver Discovery ............ ✅     ║
║ Initial Access ................. ✅     ║
║ Privilege Escalation ........... ✅     ║
║ SSH Persistence ................ ✅     ║
║ Root Verification .............. ✅     ║
╚════════════════════════════════════════╝
```

---

# 👨‍💻 Author

## Anurag Ravankar

**Cybersecurity | Penetration Testing | SOC | Linux Security**

This project is part of my hands-on cybersecurity portfolio, documenting practical experience with Linux enumeration, web exploitation, privilege escalation, evidence collection, and security reporting.

---

<div align="center">

### 🔐 Learn. Enumerate. Exploit. Document. Harden.

**Airplane — TryHackMe**

</div>

---

> **Disclaimer:** This documentation represents activity performed within an authorized TryHackMe training environment. The techniques described should not be used against systems without explicit authorization.
