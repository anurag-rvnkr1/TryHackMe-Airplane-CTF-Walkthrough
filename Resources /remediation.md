# 🛡️ Airplane — Security Findings & Remediation Guide

This document summarizes the vulnerabilities identified during the Airplane assessment and discusses defensive recommendations.

---

# Executive Summary

The attack chain demonstrates how multiple low-level weaknesses can combine into a complete system compromise.

Primary findings include:

* Local File Inclusion.
* Exposed debugger service.
* Unsafe SUID permissions.
* Wildcard sudo configuration.

---

# Finding 1 — Local File Inclusion

## Risk

High

## Impact

An attacker may read sensitive local files and gather information for further exploitation.

### Defensive Recommendations

* Validate file names using an allowlist.
* Reject directory traversal sequences.
* Store templates outside the web root.
* Disable direct filesystem access through user-controlled parameters.

---

# Finding 2 — Exposed gdbserver

## Risk

Critical

## Impact

Remote debugging services should never be exposed publicly without authentication.

### Defensive Recommendations

* Bind debugger locally.
* Require authentication.
* Restrict firewall access.
* Disable debugging in production environments.

---

# Finding 3 — Sensitive Linux Enumeration

## Risk

Medium

## Impact

Exposure of runtime process information increases attacker awareness.

### Defensive Recommendations

* Restrict unnecessary file disclosure.
* Limit web application filesystem permissions.
* Follow least privilege for service accounts.

---

# Finding 4 — SUID Misconfiguration

## Risk

High

## Impact

Privileged binaries may execute attacker-controlled commands.

### Defensive Recommendations

* Audit SUID binaries regularly.
* Remove unnecessary SUID bits.
* Use Linux capabilities where appropriate.

---

# Finding 5 — Wildcard sudo Rule

## Risk

Critical

## Impact

Improper wildcard validation can permit privilege escalation.

### Defensive Recommendations

* Avoid wildcard paths.
* Use explicit command paths.
* Validate canonical paths before execution.

---

# SSH Hardening Recommendations

* Disable password authentication where appropriate.
* Review `authorized_keys`.
* Restrict write permissions.
* Enable logging.

---

# Linux Hardening Checklist

* [ ] Remove unnecessary SUID binaries.
* [ ] Review sudoers configuration.
* [ ] Restrict debugger services.
* [ ] Patch vulnerable software.
* [ ] Validate web application inputs.
* [ ] Audit running services.
* [ ] Monitor SSH authorized keys.

---

# Blue-Team Lessons

| Offensive Observation | Defensive Control            |
| --------------------- | ---------------------------- |
| LFI enumeration       | Input validation & WAF       |
| Hidden debugger       | Service inventory monitoring |
| Reverse shell         | Egress monitoring            |
| SUID abuse            | File integrity monitoring    |
| SSH persistence       | Authorized key auditing      |
| sudo abuse            | Least privilege review       |

---

# Conclusion

The Airplane room is an excellent example of chained exploitation, showing how reconnaissance, information disclosure, exposed services, and privilege misconfigurations can combine into full root compromise. Defenders should treat each finding as part of a larger attack path rather than an isolated vulnerability.
