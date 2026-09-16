# ✈️ Airplane — TryHackMe Technical Walkthrough

> **Professional Penetration Testing Documentation**
> Author: **Anurag Ravankar**
> Platform: **TryHackMe**
> Room: **Airplane**
> Difficulty: **Medium**
> Environment: **Authorized Training Lab**

---

## Executive Summary

This document presents a professional penetration testing walkthrough for the **Airplane** room on TryHackMe. The objective was to assess a vulnerable Linux web server, gain an initial foothold through web exploitation, enumerate the operating system, identify privilege escalation vectors, and obtain root access.

The assessment follows a structured methodology inspired by the **Penetration Testing Execution Standard (PTES)** and emphasizes both offensive techniques and defensive remediation.

> **Note:** Challenge flags have been intentionally redacted to preserve academic integrity and avoid plagiarism.

---

## Table of Contents

1. Lab Overview
2. Assessment Methodology
3. Reconnaissance
4. Service Enumeration
5. Web Application Analysis
6. Local File Inclusion Discovery
7. Initial Findings

---

# 1. Lab Overview

| Property         | Value                      |
| ---------------- | -------------------------- |
| Platform         | TryHackMe                  |
| Target           | Airplane                   |
| Operating System | Linux                      |
| Difficulty       | Medium                     |
| Assessment Type  | Black-box Penetration Test |
| Goal             | Obtain Root Access         |

### Skills Covered

* Network Reconnaissance
* Web Enumeration
* Local File Inclusion (LFI)
* Linux Enumeration
* Remote Code Execution
* Privilege Escalation
* Security Reporting

---

# 2. Assessment Methodology

The assessment followed a structured attack lifecycle where each phase depended on information gathered during the previous stage.

```text
Reconnaissance
      ↓
Service Enumeration
      ↓
Web Enumeration
      ↓
LFI Exploitation
      ↓
Linux Enumeration
      ↓
Hidden Service Discovery
      ↓
Initial Shell
      ↓
Privilege Escalation
      ↓
Root Access
```

### Objectives of Each Phase

| Phase                | Objective                                                       |
| -------------------- | --------------------------------------------------------------- |
| Reconnaissance       | Identify reachable services.                                    |
| Enumeration          | Gather information about the operating system and applications. |
| Exploitation         | Obtain initial access.                                          |
| Post-Exploitation    | Enumerate users, permissions, and processes.                    |
| Privilege Escalation | Gain administrative privileges.                                 |

---

# 3. Reconnaissance

## Host Resolution

The target hostname was added to the local hosts file to simplify communication during testing.

### Why this matters

Using hostname resolution keeps commands consistent and mirrors real-world internal network assessments.

---

## Full TCP Port Scan

A full TCP scan was performed against the target.

```bash
nmap -sS -Pn -T4 -p- airplane.thm
```

### Objective

* Discover open TCP ports.
* Identify exposed attack surfaces.

### Observation

Multiple ports responded, including:

| Port | Service          |
| ---- | ---------------- |
| 22   | SSH              |
| 8000 | HTTP Application |
| 6048 | Unknown Service  |

The unknown service became important later during enumeration.

---

## Service & Version Detection

After identifying open ports, version detection was performed.

```bash
nmap -sV -sC -p22,8000,6048 airplane.thm
```

### Information Collected

* SSH server version.
* HTTP server behavior.
* Default NSE enumeration results.

### Security Observation

Port **6048** did not return a recognizable service banner, indicating a potentially hidden or custom application.

---

# 4. Service Enumeration

## HTTP Enumeration

The HTTP service running on **port 8000** became the primary attack surface.

### Initial Observations

* Dynamic web application.
* URL accepted a user-controlled parameter.
* Responses changed depending on supplied filenames.

### Testing Approach

The application was tested for:

* Directory traversal.
* File inclusion.
* Input validation weaknesses.

---

## Parameter Analysis

A file-related parameter accepted user input without proper sanitization.

Example testing methodology included requesting system files instead of expected application resources.

### Result

The server responded with local filesystem content, confirming a **Local File Inclusion (LFI)** vulnerability.

---

# 5. Local File Inclusion Discovery

## Vulnerability Description

The application allowed traversal outside its intended directory, enabling arbitrary file reads.

### Why LFI Is Dangerous

An attacker can access:

* User information.
* Configuration files.
* Process information.
* Network information.
* Credentials in some environments.

---

## Sensitive File Enumeration

The following Linux files were successfully accessed for enumeration purposes.

| File                 | Information Obtained     |
| -------------------- | ------------------------ |
| `/etc/passwd`        | Local users.             |
| `/etc/group`         | Group memberships.       |
| `/proc/self/environ` | Web process environment. |
| `/proc/net/tcp`      | Active listening ports.  |

Each file contributed additional intelligence for later exploitation.

---

## User Enumeration

Reading `/etc/passwd` revealed valid local accounts available on the target system.

### Security Impact

* Identified potential login usernames.
* Helped correlate process ownership during later enumeration.

---

## Group Enumeration

The `/etc/group` file exposed group memberships associated with discovered users.

### Why It Matters

Group membership provides clues about:

* Administrative privileges.
* Service accounts.
* Potential privilege escalation paths.

---

# 6. Initial Findings

The reconnaissance and web enumeration phase produced several valuable discoveries.

| Finding              | Severity |
| -------------------- | -------- |
| Local File Inclusion | High     |
| User Enumeration     | Medium   |
| Group Enumeration    | Medium   |
| Hidden TCP Listener  | High     |

### Key Takeaways

* Enumeration provided significantly more value than immediate exploitation.
* The LFI vulnerability exposed enough system information to continue the attack without credentials.
* Discovery of an unidentified listening port suggested an internal service requiring deeper investigation.

---

## Phase 1 Summary

### Achievements

* Identified exposed services.
* Confirmed a Local File Inclusion vulnerability.
* Enumerated Linux users and groups.
* Collected runtime process information.
* Identified a hidden TCP service for further investigation.

---

# 7. Linux Enumeration Through the `/proc` Filesystem

After confirming the Local File Inclusion vulnerability, the next objective was to collect operating system intelligence without requiring shell access. Linux exposes runtime process and networking information through the `/proc` filesystem, making it a valuable enumeration source.

---

## Enumerating the Running Web Process

The environment variables of the web application process were inspected.

### Objective

* Identify the user running the web server.
* Gather execution context for later privilege escalation.

### Observation

The environment revealed that the web application was running under a non-root account, confirming that initial access would likely have limited privileges.

**Security Impact**

Knowing the service account helps correlate process ownership during later enumeration.

---

## Enumerating Active Network Connections

The TCP socket table was inspected through the `/proc` filesystem.

### Objective

* Discover listening services not exposed through the web application.
* Identify internal ports.

### Observation

A listening socket appeared on an unfamiliar hexadecimal port. After converting the value to decimal, it matched **port 6048**, which had previously appeared during the Nmap scan.

### Why This Was Important

This confirmed that an internal service was actively listening and became the next investigation target.

---

## Process Discovery

Process information was enumerated by inspecting running process directories inside `/proc`.

### Objective

Associate the discovered listening port with the application responsible for it.

### Enumeration Strategy

* Inspect process command lines.
* Compare process identifiers with active sockets.
* Identify unusual services.

### Result

The hidden service was identified as **gdbserver**, a remote debugging service.

> **Finding:** A debugger exposed over the network significantly increases attack surface because it is designed to execute and inspect running programs.

---

# 8. Hidden Service Analysis

## What is gdbserver?

`gdbserver` is a lightweight remote debugger used during software development. It allows a debugger running on another machine to control a local process.

### Security Risk

If exposed without proper authentication, an attacker may be able to execute arbitrary code within the context of the running process.

| Observation                 | Security Impact                 |
| --------------------------- | ------------------------------- |
| Network-accessible debugger | Remote attack surface           |
| Running as a local user     | Initial foothold opportunity    |
| No visible authentication   | Potential Remote Code Execution |

---

## Exploitation Strategy

Rather than attacking SSH or the web application further, the assessment pivoted to the exposed debugger service.

### Goal

Achieve remote code execution through the debugger and obtain an interactive shell.

---

# 9. Initial Access — Remote Code Execution

A publicly documented exploitation technique for the exposed debugger service was researched and adapted to the lab environment.

### Exploitation Workflow

1. Identify compatible debugger version.
2. Prepare a reverse shell payload.
3. Start a listener on the attacker machine.
4. Trigger remote execution.

### Result

The debugger executed the payload successfully and connected back to the attacker-controlled listener.

> **Outcome:** Initial shell access was obtained as a low-privileged Linux user.

---

# 10. Reverse Shell Stabilization

The initial shell was functional but limited. A fully interactive terminal was required for reliable post-exploitation.

## Shell Upgrade

A pseudo-terminal (PTY) was spawned to improve shell usability.

### Benefits

* Interactive shell history.
* Proper terminal behavior.
* Improved privilege escalation workflow.

### Verification Checklist

* Confirm current user.
* Confirm hostname.
* Confirm working directory.
* Verify network connectivity.

The shell was now suitable for deeper enumeration.

---

# 11. Local Enumeration After Foothold

With shell access established, Linux privilege enumeration began.

## User Enumeration

The current account identity and group memberships were verified.

### Objective

Determine available permissions and identify potential escalation paths.

---

## Sudo Enumeration

The system's `sudo` configuration was inspected.

### Observation

Interesting `sudo` permissions existed but were not immediately usable from the current account.

This suggested another privilege escalation step was required before reaching root.

---

## SUID Enumeration

The filesystem was searched for binaries with the SUID permission bit enabled.

### Why SUID Matters

SUID binaries execute with the file owner's effective privileges and are a common Linux privilege escalation vector.

### Result

A writable privilege escalation path involving the **find** binary was identified.

| Enumeration Result    | Significance                   |
| --------------------- | ------------------------------ |
| SUID `find` present   | Potential privilege escalation |
| Non-root shell        | Escalation still required      |
| Interesting sudo rule | Future escalation path         |

---

# Phase 2 Summary

### Achievements

* Enumerated Linux runtime information through `/proc`.
* Identified the hidden `gdbserver` service.
* Achieved Remote Code Execution.
* Obtained an interactive reverse shell.
* Enumerated SUID binaries and potential privilege escalation vectors.

### Security Findings

| Finding                            | Severity |
| ---------------------------------- | -------- |
| Exposed `gdbserver` Service        | Critical |
| Information Disclosure via `/proc` | High     |
| SUID Binary Available              | High     |

---

# 12. Privilege Escalation — SUID Binary Abuse

After obtaining a stable shell, the next objective was to escalate privileges beyond the compromised user account.

---

## Understanding SUID

The **Set User ID (SUID)** permission allows a binary to execute with the privileges of its owner rather than the user executing it.

### Why This Matters

Improperly configured SUID binaries can permit command execution with elevated privileges and are frequently targeted during Linux privilege escalation assessments.

---

## Enumerating SUID Binaries

The filesystem was searched for binaries with the SUID permission bit enabled.

```bash
find / -perm -4000 -type f 2>/dev/null
```

### Observation

Several standard Linux binaries were identified. One entry stood out because it could be abused to execute arbitrary commands.

### Analysis

The identified binary appeared in GTFOBins, indicating a known privilege escalation technique.

---

## Exploiting the Binary

The documented GTFOBins method was adapted to the target environment.

### Result

Command execution was achieved with elevated privileges, providing access to another user account on the system.

### Security Impact

This demonstrates why SUID permissions should be reviewed regularly and removed whenever they are not strictly required.

---

# 13. Lateral User Access

Following the successful SUID exploitation, access was obtained to a higher-privileged user account.

### Objectives

* Enumerate the new user environment.
* Review available credentials and keys.
* Search for additional escalation opportunities.

### Observation

The user account possessed greater system access and contained information useful for maintaining persistence.

---

# 14. SSH Key Persistence

## Why Persistence Was Established

Reverse shells are often unstable and can terminate unexpectedly. To ensure reliable access during the assessment, SSH key-based authentication was configured.

---

## Generating a Key Pair

An SSH key pair was generated on the attacker system.

### Workflow

1. Generate public/private key pair.
2. Add public key to the target user's `authorized_keys`.
3. Authenticate using the private key.

### Benefits

| Reverse Shell       | SSH Session     |
| ------------------- | --------------- |
| Temporary           | Persistent      |
| Limited interaction | Full terminal   |
| Less stable         | Reliable access |

---

## Verification

SSH authentication succeeded, providing a fully interactive shell under the target user's account.

### Outcome

The assessment could continue without relying on reverse shell connectivity.

---

# 15. Root Privilege Enumeration

With persistent access established, attention shifted toward obtaining administrative privileges.

---

## Reviewing sudo Permissions

The user's sudo privileges were inspected.

```bash
sudo -l
```

### Observation

A Ruby-based script could be executed through sudo under specific conditions.

### Security Concern

The command contained a wildcard pattern that could be manipulated through path traversal techniques.

---

# 16. Wildcard sudo Misconfiguration

## Background

Administrators sometimes use wildcard characters inside sudo rules to simplify access control.

Example concept:

```text
/user/scripts/*
```

Although convenient, wildcard rules may behave unexpectedly when filesystem path normalization is involved.

---

## Vulnerability Analysis

The configured rule trusted user-controlled paths without fully validating their canonical location.

### Risk

An attacker may supply crafted paths that satisfy the wildcard check while ultimately resolving to unintended files.

### Security Impact

This transforms a seemingly restricted sudo rule into a privilege escalation vector.

---

## Exploitation

A controlled path traversal technique was used to execute an attacker-selected file through the permitted sudo command.

### Result

The command executed successfully with root privileges.

### Outcome

Administrative access to the target system was achieved.

---

# 17. Root Access Verification

After successful exploitation, system privileges were verified.

### Validation Steps

* Confirm current user identity.
* Verify effective user ID.
* Confirm access to restricted files.
* Verify root shell functionality.

### Result

The system returned:

```text
uid=0(root)
```

indicating full administrative control of the target host.

---

# 18. Flag Verification

The room objectives were completed successfully.

To preserve the educational value of the challenge, flags are intentionally redacted.

```text
user.txt → THM{********************************}

root.txt → THM{********************************}
```

### Repository Policy

This project focuses on:

* Methodology
* Enumeration
* Exploitation workflow
* Defensive lessons

rather than publishing challenge answers.

---

# Phase 3 Summary

### Achievements

✅ Privilege escalation through SUID abuse

✅ Access to higher-privileged user account

✅ SSH persistence established

✅ sudo misconfiguration identified

✅ Wildcard bypass successfully exploited

✅ Root access obtained

---

## Security Findings

| Finding                      | Severity |
| ---------------------------- | -------- |
| Unsafe SUID Binary           | High     |
| SSH Persistence Opportunity  | Medium   |
| Wildcard sudo Rule           | Critical |
| Privilege Escalation to Root | Critical |

### Key Takeaways

* SUID permissions require regular auditing.
* Persistent access methods improve assessment reliability.
* Wildcard sudo rules can introduce unintended privilege escalation paths.
* Small configuration mistakes can combine into complete system compromise.

---

# 19. MITRE ATT&CK Mapping

This assessment aligns the observed attack techniques with the **MITRE ATT&CK Framework**, providing a defensive perspective on the exploitation chain.

| MITRE Tactic             | Technique                                 | Purpose                                                  |
| ------------------------ | ----------------------------------------- | -------------------------------------------------------- |
| **Initial Access**       | T1190 – Exploit Public-Facing Application | Local File Inclusion vulnerability.                      |
| **Discovery**            | T1082 – System Information Discovery      | Operating system and user enumeration.                   |
| **Discovery**            | T1057 – Process Discovery                 | Enumerating running processes via `/proc`.               |
| **Discovery**            | T1046 – Network Service Discovery         | Identifying hidden listening services.                   |
| **Execution**            | T1059 – Command and Scripting Interpreter | Executing commands through the exposed debugger service. |
| **Persistence**          | T1098.004 – SSH Authorized Keys           | Establishing persistent authenticated access.            |
| **Privilege Escalation** | T1548 – Abuse Elevation Control Mechanism | Exploiting SUID binaries and `sudo` misconfiguration.    |

**Defensive Insight**

Mapping offensive actions to ATT&CK techniques helps defenders understand how attackers progress through a compromise and where monitoring or prevention controls should be placed.

---

# 20. Security Findings Summary

The assessment identified multiple vulnerabilities that combined into a complete system compromise.

| Finding                            | Severity    | Recommendation                                       |
| ---------------------------------- | ----------- | ---------------------------------------------------- |
| Local File Inclusion               | 🔴 High     | Validate and whitelist user-controlled file paths.   |
| Exposed `gdbserver` Service        | 🔴 Critical | Disable remote debugging or restrict network access. |
| Information Disclosure via `/proc` | 🟠 High     | Restrict unnecessary filesystem exposure.            |
| Unsafe SUID Binary                 | 🟠 High     | Audit and remove unnecessary SUID permissions.       |
| Wildcard `sudo` Configuration      | 🔴 Critical | Replace wildcard rules with explicit command paths.  |

---

# 21. Defensive Recommendations

The vulnerabilities discovered during this room demonstrate several common Linux hardening principles.

## Web Application Security

* Validate user input before accessing filesystem resources.
* Prevent directory traversal using allowlists.
* Avoid exposing sensitive files through application parameters.

## Linux Hardening

* Audit SUID binaries regularly.
* Remove unused privileged binaries.
* Apply the principle of least privilege for service accounts.

## Service Security

* Never expose debugger services in production environments.
* Restrict internal services using firewall rules.
* Monitor unexpected listening ports.

## SSH Security

* Protect `authorized_keys` permissions.
* Monitor new SSH keys.
* Disable unused authentication methods where appropriate.

---

# 22. Lessons Learned

This room reinforced several important penetration testing concepts.

### Technical Skills

* Performing structured reconnaissance before exploitation.
* Using Local File Inclusion for information disclosure.
* Leveraging the Linux `/proc` filesystem for process and network enumeration.
* Identifying hidden services through system analysis.
* Escalating privileges using legitimate Linux features.

### Reporting Skills

* Recording observations during every assessment phase.
* Separating findings from exploitation steps.
* Providing remediation alongside vulnerabilities.
* Organizing screenshots and evidence professionally.

---

# 23. Assessment Outcome

| Objective                      | Status |
| ------------------------------ | ------ |
| Reconnaissance Completed       | ✅      |
| Services Enumerated            | ✅      |
| LFI Identified                 | ✅      |
| Initial Shell Obtained         | ✅      |
| Privilege Escalation Completed | ✅      |
| Root Access Achieved           | ✅      |
| Findings Documented            | ✅      |

---

# 24. Conclusion

The **Airplane** room provides a practical demonstration of how multiple seemingly independent weaknesses can be chained together into a complete system compromise.

The assessment began with **network reconnaissance**, progressed through **web application exploitation** using Local File Inclusion, leveraged **Linux process enumeration** to discover a hidden debugger service, achieved **Remote Code Execution**, and concluded with **multiple privilege escalation techniques** leading to root access.

Beyond exploitation, this project emphasizes professional cybersecurity documentation by including methodology, findings, MITRE ATT&CK mapping, remediation recommendations, and evidence organization. The repository is intended to showcase both **technical penetration testing skills** and the ability to produce clear, structured security reports suitable for a cybersecurity portfolio.

---

# References

* TryHackMe — Airplane Room
* MITRE ATT&CK Framework
* GTFOBins
* HackTricks Linux Privilege Escalation
* OWASP Path Traversal & Local File Inclusion Documentation
* Linux Manual Pages (`proc`, `sudo`, `ssh`)

---

## Repository Notes

* **Platform:** TryHackMe (Authorized Lab)
* **Documentation Type:** Educational Penetration Testing Report
* **Flags:** Intentionally redacted for academic integrity.
* **Purpose:** Portfolio demonstration and cybersecurity learning.
