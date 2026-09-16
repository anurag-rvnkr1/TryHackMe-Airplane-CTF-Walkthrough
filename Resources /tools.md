# 🛠️ Airplane — Tool Reference

This document explains **why each tool was used**, not just the commands.

---

# Tool Stack

| Tool         | Role                           |
| ------------ | ------------------------------ |
| Nmap         | Reconnaissance                 |
| Burp Suite   | Web testing                    |
| Searchsploit | Exploit research               |
| Python       | Script execution               |
| msfvenom     | Payload generation             |
| Netcat       | Reverse shell listener         |
| SSH          | Authenticated access           |
| GTFOBins     | Privilege escalation reference |

---

# Nmap

## Purpose

* Discover open ports.
* Identify services.
* Perform version detection.
* Execute NSE scripts.

### Learning Outcome

Nmap provides the initial attack surface map.

---

# Burp Suite

## Purpose

Intercept and modify HTTP requests.

### Why It Was Useful

Allowed controlled testing of vulnerable parameters without relying on browser behavior.

---

# Searchsploit

## Purpose

Search the Exploit-DB database locally.

### Learning Outcome

Useful for quickly identifying publicly documented vulnerabilities affecting discovered services.

---

# Python

## Purpose

* Execute enumeration scripts.
* Run exploit scripts.
* Upgrade shell.

---

# msfvenom

## Purpose

Generate payloads compatible with Linux reverse TCP shells.

### Learning Outcome

Understand payload creation without requiring a full Metasploit session.

---

# Netcat

## Purpose

Receive incoming reverse shell connections.

### Learning Outcome

Simple listener for post-exploitation access.

---

# SSH

## Purpose

Persistent authenticated shell access using key-based authentication.

### Benefits

* Stable terminal.
* Secure authentication.
* Better usability than reverse shell.

---

# GTFOBins

## Purpose

Research legitimate Unix binaries that can be abused during privilege escalation.

### Learning Outcome

Helps identify privilege escalation opportunities from existing binaries instead of custom exploits.

---

# Recommended Enumeration Workflow

1. Nmap
2. Browser + Burp Suite
3. Linux Enumeration
4. Searchsploit
5. GTFOBins
6. SSH
7. Documentation

---

# Defensive Perspective

Each tool can also support blue-team validation:

| Tool         | Defensive Use              |
| ------------ | -------------------------- |
| Nmap         | Asset discovery            |
| Burp Suite   | Secure testing             |
| GTFOBins     | Hardening audits           |
| SSH          | Access validation          |
| Searchsploit | Vulnerability verification |
