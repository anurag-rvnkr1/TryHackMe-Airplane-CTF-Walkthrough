# 📝 Airplane — Technical Notes & Learning Journal

> Personal learning notes collected while completing the **TryHackMe Airplane** room.

---

# Purpose

This document summarizes important concepts, observations, and lessons learned during the assessment. It is intentionally written as a study reference rather than a walkthrough.

---

# Key Concepts Learned

## 1. Local File Inclusion (LFI)

**Definition**

Local File Inclusion occurs when an application allows user-controlled file paths to be read from the server without proper validation.

**Indicators**

* User-controlled file parameter.
* Relative path traversal (`../`).
* Server returns local operating system files.

**Why it mattered**

LFI became the primary foothold for collecting sensitive information without authentication.

---

## 2. Linux Enumeration Through `/proc`

The `/proc` filesystem exposes runtime information about processes and networking.

Useful locations discovered during this room:

| File                  | Purpose                                           |
| --------------------- | ------------------------------------------------- |
| `/proc/self/environ`  | Environment variables of the running web process. |
| `/proc/net/tcp`       | Active TCP listeners and sockets.                 |
| `/proc/<PID>/cmdline` | Command line arguments of running processes.      |

### Lesson

The `/proc` filesystem is often overlooked but can reveal internal services that are not visible from the web application itself.

---

## 3. Hexadecimal Port Conversion

Entries inside `/proc/net/tcp` store ports in hexadecimal.

Example workflow:

1. Read hexadecimal port.
2. Convert to decimal.
3. Match with Nmap results.

This helps identify hidden services owned by discovered processes.

---

## 4. Process Discovery

Instead of relying only on Nmap:

* Enumerate active sockets.
* Identify process ownership.
* Correlate PID with command line.

This technique exposed the hidden debugger service.

---

## 5. gdbserver Exposure

`gdbserver` is designed for remote debugging.

### Security Lesson

A debugger exposed on a network interface dramatically increases attack surface because it may permit remote execution if left unauthenticated.

---

## 6. Reverse Shell Stabilization

After obtaining a shell:

* Upgrade to an interactive TTY.
* Verify user identity.
* Enumerate permissions before privilege escalation.

Stable shells improve usability during Linux post-exploitation.

---

## 7. Effective UID vs Real UID

A SUID binary changes the **Effective UID**, but not necessarily the **Real UID**.

### Why this matters

Some privilege checks (such as `sudo`) still rely on the real user identity.

Understanding UID behavior explains why additional privilege escalation steps became necessary.

---

## 8. SSH Key Persistence

Adding a public key to `authorized_keys` creates authenticated shell access.

### Advantages

* Stable login.
* Interactive terminal.
* Avoids unreliable reverse shell sessions.

---

## 9. Wildcard `sudo` Misconfiguration

A wildcard inside `sudoers` can become dangerous when path normalization is possible.

### Lesson

Security controls must validate canonical filesystem paths instead of trusting wildcard patterns.

---

# Enumeration Checklist Used

* [x] Host resolution
* [x] TCP port scan
* [x] Service detection
* [x] Web parameter inspection
* [x] Sensitive file enumeration
* [x] Process enumeration
* [x] Socket enumeration
* [x] Privilege enumeration
* [x] SUID discovery
* [x] `sudo` permission review

---

# Commands Worth Remembering

## Network Enumeration

```bash
nmap -sS -Pn -T4 -p- TARGET
nmap -sV -sC TARGET
```

## Linux Enumeration

```bash
id
whoami
hostname
sudo -l
find / -perm -4000 2>/dev/null
```

## Shell Upgrade

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```

---

# Personal Takeaways

* Enumeration is more valuable than rushing exploitation.
* `/proc` can reveal hidden attack paths.
* SUID binaries should always be audited.
* Wildcard `sudo` rules can introduce privilege escalation.
* Proper documentation is part of professional penetration testing.
