# Room 1 — OWASP Top 10 2025: IAAA Failures

🔗 **Room Link:** [https://tryhackme.com/room/owasptopten2025one](https://tryhackme.com/room/owasptopten2025one)  
📊 **Difficulty:** Beginner  
🏷️ **Tags:** Access Control, Authentication, Logging

---

## 📖 Room Introduction

This room introduces the **IAAA framework** — a sequential model used to verify users and their actions within a web application. IAAA stands for:

| Component | Meaning |
|-----------|---------|
| **I**dentification | Who are you? (username, email) |
| **A**uthentication | Prove it (password, MFA) |
| **A**uthorisation | What are you allowed to do? |
| **A**ccountability | Log what you did |

The room covers **3 OWASP 2025 categories** that represent failures at different points in the IAAA chain:

- **A01: Broken Access Control** — Authorisation failures (users doing things they shouldn't)
- **A07: Authentication Failures** — Authentication weaknesses (brute-force, weak passwords)
- **A09: Logging & Alerting Failures** — Accountability gaps (attacks happening silently)

---

## 🎯 What is the Goal?

By the end of this room, you should understand:

1. What happens when access control is improperly enforced
2. How attackers exploit authentication weaknesses
3. Why proper logging is critical for detecting and responding to attacks
4. How to identify **horizontal vs. vertical privilege escalation**

---

## 🛠️ What I Did & How I Solved It

### Task 1 — Introduction
Read through the introductory material on the IAAA model. No practical challenge here — just building context.

---

### Task 2 — A01: Broken Access Control

**Concept:**  
Broken Access Control occurs when a user can perform actions or access data beyond their intended permissions. There are two types:
- **Horizontal Privilege Escalation** — accessing another user's data at the same privilege level
- **Vertical Privilege Escalation** — gaining higher-privilege access (e.g., becoming an admin)

**What I Did:**
- Accessed the deployed web application
- Explored the user profile page as a normal user
- Noticed that user data was accessible by directly manipulating URL parameters (e.g., changing `user_id=1` to `user_id=2`)
- This allowed viewing another user's data without any authorisation check — classic **horizontal privilege escalation**

**Challenge Q&A:**
> *If you don't get access to more roles but can view the data of another user, what type of privilege escalation is this?*  
> ✅ **Horizontal Privilege Escalation**

**Key Technique Used:**  
Manually modifying URL/API parameters to access unauthorized resources (IDOR — Insecure Direct Object Reference).

---

### Task 3 — A07: Authentication Failures

**Concept:**  
Authentication failures include weak passwords, missing brute-force protection, insecure credential storage, and broken session management.

**What I Did:**
- Identified a login form with no rate limiting or lockout mechanism
- Used a wordlist-based brute-force approach against the login page
- Successfully authenticated as another user by guessing the password
- Examined session tokens to verify authentication was granted

**Challenge Q&A:**
> *What is the IP of the attacker (from logs)?*  
> ✅ Found in the log analysis task by filtering for repeated failed login attempts from a single IP

> *What is the username associated with the compromised account?*  
> ✅ Found by identifying the first successful login after a series of failures from the attacker IP

**Key Technique Used:**  
Log analysis to reconstruct a brute-force attack timeline; identifying the victim account from authentication logs.

---

### Task 4 — A09: Logging & Alerting Failures

**Concept:**  
If applications don't log security-relevant events (logins, failed attempts, access to sensitive data), attacks go undetected. Even when logs exist, they're useless without alerts and monitoring.

**What I Did:**
- Examined provided application log files
- Filtered through log entries to identify suspicious patterns
- Located the attacker's IP address by spotting repeated `401 Unauthorized` responses followed by a `200 OK`
- Identified the compromised username from the successful login entry

**Key Observation:**  
The attack was only detectable *because* there were logs — the room highlighted how many applications skip logging entirely, making forensic investigation impossible.

---

## 💡 What I Learned

### Technical Takeaways

| Vulnerability | What I Learned |
|---------------|---------------|
| **Broken Access Control (A01)** | Never trust client-supplied IDs; always verify server-side that the requesting user has rights to that specific resource. Implement object-level authorization checks on every request. |
| **Authentication Failures (A07)** | Implement account lockout, CAPTCHA, or exponential backoff after failed login attempts. Use strong password policies and MFA where possible. |
| **Logging & Alerting Failures (A09)** | Log all authentication events (success AND failure) with timestamps and IP addresses. Set up alerts for anomalous patterns like >5 failed logins from one IP. |

### Conceptual Takeaways

- The **IAAA model is sequential** — you can't skip steps. If authentication is bypassed, authorisation is meaningless.
- **Horizontal privilege escalation** is often overlooked because it doesn't involve becoming an admin — but leaking another user's data is just as damaging.
- **Log analysis is a real security skill.** Being able to reconstruct attack timelines from logs is essential for both defenders and penetration testers.
- **IDOR (Insecure Direct Object Reference)** is one of the most common and impactful access control flaws in real-world applications.

### Tools/Concepts Encountered
- Manual URL parameter manipulation
- Log file analysis
- Brute-force authentication concepts
- HTTP status codes (401, 200) as attack indicators

---
