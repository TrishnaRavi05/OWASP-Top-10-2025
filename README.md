# OWASP-Top-10-2025
Try Hack Me Room
## 📖 Introduction

The **OWASP Top 10** is a globally recognized standard document published by the **Open Web Application Security Project (OWASP)** — a non-profit foundation dedicated to improving software security. The list identifies the **10 most critical security risks** to web applications and is widely used by developers, security teams, and organizations as a baseline for securing software.

The **2025 edition** of the OWASP Top 10 reflects the evolving threat landscape, incorporating new risks around supply chain security, AI-assisted attacks, and modern API threats.

**TryHackMe's OWASP Top 10 (2025) module** brings this list to life through hands-on rooms, letting you exploit real vulnerabilities in a safe, guided environment. It's designed for beginners and assumes no prior security knowledge.

---

## 🎯 Goal of this Module

The primary goals of the TryHackMe OWASP Top 10 (2025) module are:

- ✅ Understand each of the **10 critical web application vulnerabilities** in the 2025 OWASP list
- ✅ Learn **how vulnerabilities occur** — the root cause and real-world context
- ✅ **Exploit** vulnerabilities in a safe, controlled lab environment
- ✅ Understand **how to prevent and mitigate** each vulnerability
- ✅ Build a solid foundation in **web application security**
- ✅ Apply theoretical knowledge through **practical challenges and CTF-style flags**

By the end of the module, you should be able to recognize and explain all 10 OWASP vulnerability categories, understand attacker thinking, and apply defensive techniques.

---

## 🗂️ Rooms Available

The module is split into **3 rooms**, each covering a cluster of related OWASP 2025 categories:

| # | Room Name | OWASP Categories Covered | Difficulty | Link |
|---|-----------|--------------------------|------------|------|
| 1 | **IAAA Failures** | A01: Broken Access Control, A07: Authentication Failures, A09: Logging & Alerting Failures | Beginner | [Room 1](https://tryhackme.com/room/owasptopten2025one) |
| 2 | **Application Design Flaws** | A02: Security Misconfigurations, A03: Software Supply Chain Failures, A06: Insecure Design, A10: SSRF | Beginner | [Room 2](https://tryhackme.com/room/owasptopten2025two) |
| 3 | **Insecure Data Handling** | A04: Cryptographic Failures, A05: Injection, A08: Software or Data Integrity Failures | Beginner | [Room 3](https://tryhackme.com/room/owasptopten2025three) |

> 📌 Module Page: [https://tryhackme.com/module/owasp-top-10-2025](https://tryhackme.com/module/owasp-top-10-2025)

---

## 📋 The Full OWASP Top 10 — 2025 List

| Rank | Category | Description |
|------|----------|-------------|
| A01 | Broken Access Control | Users accessing resources/actions beyond their permissions |
| A02 | Security Misconfiguration | Unsafe defaults, open debug settings, exposed services |
| A03 | Software Supply Chain Failures | Compromised/outdated/unverified third-party components |
| A04 | Cryptographic Failures | Weak/missing encryption, poor key management |
| A05 | Injection | SQL, SSTI, command injection via unsanitized input |
| A06 | Insecure Design | Flawed architecture and logic at the design level |
| A07 | Authentication Failures | Weak auth, brute-force susceptibility, session issues |
| A08 | Software or Data Integrity Failures | Insecure deserialization, CI/CD pipeline attacks |
| A09 | Logging & Alerting Failures | Missing/inadequate logs making attacks invisible |
| A10 | Server-Side Request Forgery (SSRF) | Server tricked into making requests to internal services |

---
