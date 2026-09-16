# ✈️ Airplane — TryHackMe Penetration Testing Report

<p align="center">
  <img src="assets/banner.png" width="100%" alt="Airplane TryHackMe Banner"/>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/TryHackMe-Airplane-red?style=for-the-badge&logo=tryhackme"/>
  <img src="https://img.shields.io/badge/Linux-Penetration%20Testing-FCC624?style=for-the-badge&logo=linux&logoColor=black"/>
  <img src="https://img.shields.io/badge/Difficulty-Medium-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Documentation-GitHub%20Pages-blue?style=for-the-badge"/>
</p>

<p align="center">
Professional documentation of the <strong>Airplane</strong> TryHackMe room following a penetration testing methodology from reconnaissance to root privilege escalation.
</p>

---

# 📑 Table of Contents

* Executive Summary
* Lab Overview
* Attack Chain
* Assessment Methodology
* Reconnaissance
* Web Enumeration
* Local File Inclusion
* Linux Enumeration
* Hidden Service Discovery
* Initial Access
* Privilege Escalation
* Root Verification
* MITRE ATT&CK Mapping
* Security Findings
* Remediation
* Lessons Learned
* References

---

# Executive Summary

The **Airplane** room is a Linux-based Capture The Flag challenge on TryHackMe that simulates a realistic web application penetration test. The assessment demonstrates how multiple vulnerabilities—including **Local File Inclusion (LFI)**, an exposed **gdbserver** instance, **SUID misconfiguration**, and an insecure **sudo wildcard rule**—can be chained into complete administrative compromise. <Cite ref={["turn0search0","turn0search3"]}/>

This report documents the complete methodology used during the assessment while intentionally **redacting challenge flags** to preserve academic integrity.

---

# Lab Overview

| Property         | Value                      |
| ---------------- | -------------------------- |
| Platform         | TryHackMe                  |
| Room             | Airplane                   |
| Operating System | Linux                      |
| Difficulty       | Medium                     |
| Assessment Type  | Black-box Penetration Test |
| Objective        | Obtain Root Access         |

### Skills Demonstrated

* Network Reconnaissance
* Service Enumeration
* Local File Inclusion
* Linux Enumeration
* Process Discovery
* Remote Code Execution
* Reverse Shell Stabilization
* Privilege Escalation
* SSH Persistence
* Security Reporting

---

# Attack Chain Overview

```text
Reconnaissance
      │
      ▼
Nmap Enumeration
      │
      ▼
HTTP Service Discovery
      │
      ▼
Local File Inclusion
      │
      ▼
/proc Enumeration
      │
      ▼
Hidden gdbserver
      │
      ▼
Remote Code Execution
      │
      ▼
Reverse Shell
      │
      ▼
SUID Privilege Escalation
      │
      ▼
SSH Persistence
      │
      ▼
Wildcard sudo Exploit
      │
      ▼
ROOT ACCESS
```

---

# Assessment Methodology

This assessment follows a structured penetration testing lifecycle inspired by **PTES**.

| Phase                | Objective                                    |
| -------------------- | -------------------------------------------- |
| Reconnaissance       | Discover exposed services.                   |
| Enumeration          | Gather application and OS intelligence.      |
| Exploitation         | Gain an initial foothold.                    |
| Post Exploitation    | Enumerate users, permissions, and processes. |
| Privilege Escalation | Obtain administrative privileges.            |
| Reporting            | Document findings and remediation.           |

---

# 1. Reconnaissance

## Host Resolution

The target hostname was added to the local hosts file so browser redirections and tools could resolve `airplane.thm` correctly.

<img src="../Screenshots/figure-1-hosts-file.png" width="95%">

**Figure 1.** Host resolution configuration using `/etc/hosts`.

---

## Full TCP Port Scan

A complete TCP SYN scan was performed.

```bash
nmap -sS -Pn -T4 -p- airplane.thm
```

<img src="../Screenshots/figure-2-nmap-scan.png" width="95%">

**Figure 2.** Full TCP port scan identifying SSH, HTTP, and an unknown service.

### Open Ports

| Port | Service |
| ---- | ------- |
| 22   | SSH     |
| 6048 | Unknown |
| 8000 | HTTP    |

The unidentified port later became the primary exploitation target.

---

## Service Enumeration

Service and version detection provided additional intelligence.

```bash
nmap -sV -sC -p22,6048,8000 airplane.thm
```

<img src="../Screenshots/figure-3-service-enumeration.png" width="95%">

**Figure 3.** Service and version detection results.

### Findings

* OpenSSH service.
* Werkzeug Python HTTP server.
* Unknown listener on port **6048**.

---

# 2. Web Enumeration

The web application on port **8000** accepted a `page=` parameter that dynamically loaded HTML content.

## Parameter Analysis

The parameter was tested for traversal behavior.

<img src="../Screenshots/figure-4-lfi-discovery.png" width="95%">

**Figure 4.** Discovery of Local File Inclusion through the `page` parameter.

### Observation

The application allowed traversal outside its intended directory, confirming a **Local File Inclusion vulnerability**.

---

# 3. Local File Inclusion

## Enumerating `/etc/passwd`

The LFI vulnerability exposed sensitive Linux files.

<img src="../Screenshots/figure-5-passwd-enumeration.png" width="95%">

**Figure 5.** Reading `/etc/passwd` through the vulnerable endpoint.

### Intelligence Gathered

* Valid Linux usernames.
* Home directories.
* Shell information.

This information became valuable during later privilege escalation.

---

# 4. Linux Enumeration

After confirming LFI, Linux runtime information was gathered through `/proc`.

## Enumerating Network Connections

<img src="../Screenshots/figure-6-proc-net-tcp.png" width="95%">

**Figure 6.** Enumerating `/proc/net/tcp` to identify hidden listening sockets.

### Discovery

A hexadecimal TCP listener corresponded to **port 6048**.

### Why It Matters

Internal services often expose additional attack surfaces unavailable through normal web enumeration.

---

## Process Enumeration

The `/proc` filesystem was used to correlate listening ports with running processes.

<img src="../Screenshots/figure-7-gdbserver-discovery.png" width="95%">

**Figure 7.** Process enumeration identifying `gdbserver`.

### Finding

A remote debugger service was exposed without authentication.

---

# 5. Hidden Service Analysis

## gdbserver

`gdbserver` is a remote debugging service intended for development environments.

### Security Risk

| Observation           | Impact                           |
| --------------------- | -------------------------------- |
| Network Accessible    | Remote attack surface.           |
| Runs under local user | Initial foothold opportunity.    |
| No authentication     | Potential Remote Code Execution. |

---

# 6. Initial Access

## Remote Code Execution

A compatible exploitation technique was used against the exposed debugger service.

<img src="../Screenshots/figure-8-reverse-shell.png" width="95%">

**Figure 8.** Successful reverse shell connection.

### Outcome

* Initial shell obtained.
* Low-privileged user access established.

---

## Reverse Shell Stabilization

A PTY shell was spawned for improved interaction.

### Benefits

* Interactive terminal.
* Command history.
* Stable privilege escalation workflow.

---

# 7. Privilege Escalation

## Enumerating SUID Binaries

The filesystem was searched for SUID-enabled binaries.

```bash
find / -perm -4000 -type f 2>/dev/null
```

<img src="../Screenshots/figure-9-suid-enumeration.png" width="95%">

**Figure 9.** Enumeration of SUID binaries.

### Observation

A SUID-enabled `find` binary was available.

---

## GTFOBins Privilege Escalation

The documented GTFOBins technique was applied.

<img src="../Screenshots/figure-10-suid-find-privesc.png" width="95%">

**Figure 10.** SUID `find` privilege escalation.

### Result

Command execution with elevated privileges allowed access to another local account.

---

# 8. SSH Persistence

Reverse shells were replaced with authenticated SSH access.

<img src="../Screenshots/figure-11-ssh-persistence.png" width="95%">

**Figure 11.** Successful SSH key authentication.

### Advantages

| Reverse Shell | SSH Session |
| ------------- | ----------- |
| Temporary     | Persistent  |
| Limited       | Interactive |
| Unstable      | Reliable    |

---

# 9. Root Escalation

## Reviewing sudo Permissions

`sudo -l` revealed a wildcard rule permitting privileged execution.

### Vulnerability

A wildcard path inside `sudoers` trusted attacker-controlled filesystem paths.

---

## Wildcard sudo Exploitation

<img src="../Screenshots/figure-12-sudo-exploit.png" width="95%">

**Figure 12.** Exploiting the wildcard `sudo` misconfiguration.

### Security Impact

The misconfigured rule permitted privileged execution outside the intended directory.

---

# 10. Root Verification

Administrative privileges were verified after successful exploitation.

<img src="../Screenshots/figure-13-root-shell.png" width="95%">

**Figure 13.** Root shell verification.

### Verification Commands

* `whoami`
* `id`
* `hostname`

### Result

```text
uid=0(root)
gid=0(root)
groups=0(root)
```

Root access was successfully obtained.

---

# Flag Verification

To preserve the educational integrity of the TryHackMe room, challenge flags have been intentionally redacted.

```text
user.txt → THM{********************************}

root.txt → THM{********************************}
```

---

# MITRE ATT&CK Mapping

| Tactic               | Technique                                 |
| -------------------- | ----------------------------------------- |
| Initial Access       | T1190 – Exploit Public-Facing Application |
| Discovery            | T1082 – System Information Discovery      |
| Discovery            | T1057 – Process Discovery                 |
| Discovery            | T1046 – Network Service Discovery         |
| Execution            | T1059 – Command Interpreter               |
| Persistence          | T1098.004 – SSH Authorized Keys           |
| Privilege Escalation | T1548 – Abuse Elevation Control Mechanism |

---

# Security Findings

| Finding                | Severity    |
| ---------------------- | ----------- |
| Local File Inclusion   | 🔴 High     |
| Exposed gdbserver      | 🔴 Critical |
| Information Disclosure | 🟠 High     |
| Unsafe SUID Binary     | 🟠 High     |
| Wildcard sudo          | 🔴 Critical |

---

# Remediation Recommendations

## Web Application

* Validate file paths.
* Prevent traversal sequences.
* Implement allowlists.

## Linux Hardening

* Audit SUID binaries.
* Remove unnecessary privileges.
* Apply least privilege.

## Service Security

* Disable remote debugging.
* Restrict internal listeners.
* Monitor unexpected services.

## SSH Hardening

* Protect `authorized_keys`.
* Monitor new keys.
* Disable unused authentication methods.

---

# Lessons Learned

### Offensive Security

* Enumeration drives exploitation.
* `/proc` provides valuable runtime intelligence.
* Debug services should never be exposed publicly.

### Defensive Security

* Misconfigurations often chain together.
* Monitoring privileged binaries reduces attack surface.
* Secure sudo configuration is essential.

---

# Assessment Outcome

| Objective            | Status |
| -------------------- | ------ |
| Reconnaissance       | ✅      |
| Enumeration          | ✅      |
| Initial Access       | ✅      |
| Privilege Escalation | ✅      |
| Root Access          | ✅      |
| Documentation        | ✅      |

---

# References

* TryHackMe — Airplane Room.
* MITRE ATT&CK Framework.
* GTFOBins.
* HackTricks Linux Privilege Escalation.
* OWASP Path Traversal & Local File Inclusion.

Additional learning resources were consulted for methodology and Linux privilege escalation concepts. <Cite ref={["turn0search0","turn0search3","turn0search6"]}/>

---

# About This Portfolio Project

This repository demonstrates a complete penetration testing workflow performed within an **authorized TryHackMe laboratory**. The documentation is structured as a professional security assessment report and is intended to showcase practical Linux exploitation, privilege escalation, and cybersecurity reporting skills for portfolio and educational purposes.

---

<p align="center">
  <strong>© Anurag Ravankar • Cybersecurity Portfolio Project</strong>
</p>
