# Room 2 — OWASP Top 10 2025: Application Design Flaws

🔗 **Room Link:** [https://tryhackme.com/room/owasptopten2025two](https://tryhackme.com/room/owasptopten2025two)  
📊 **Difficulty:** Beginner  
🏷️ **Tags:** Misconfiguration, Supply Chain, Insecure Design, SSRF

---

## 📖 Room Introduction

This room focuses on vulnerabilities that stem from **flawed architecture and system design decisions** — mistakes made before a single line of application code is written. These are often the most dangerous class of vulnerability because they're baked into the foundation of a system and hard to patch retroactively.

The room covers **4 OWASP 2025 categories**:

| OWASP ID | Category | Description |
|----------|----------|-------------|
| **A02** | Security Misconfiguration | Unsafe defaults, debug mode, exposed admin panels |
| **A03** | Software Supply Chain Failures | Compromised or outdated third-party components |
| **A06** | Insecure Design | Architectural flaws and missing security controls by design |
| **A10** | Server-Side Request Forgery (SSRF) | Server tricked into making requests to internal/unintended targets |

---

## 🎯 What is the Goal?

By the end of this room, you should understand:

1. How misconfigured systems expose attack surfaces even with clean code
2. Why third-party dependencies are a major risk vector
3. The difference between a *bug* and a *design flaw* — and why design flaws are harder to fix
4. How SSRF allows attackers to pivot to internal infrastructure

---

## 🛠️ What I Did & How I Solved It

### Task 1 — Introduction
Read the room intro explaining that not all vulnerabilities come from bad code — many come from bad *decisions* made at the architecture and configuration level.

---

### Task 2 — A02: Security Misconfiguration

**Concept:**  
Security misconfigurations happen when systems are deployed with:
- Unsafe default settings (default admin credentials)
- Debug mode left on in production
- Exposed admin panels or unnecessary services
- Missing security headers
- Verbose error messages that reveal stack traces

**What I Did:**
- Interacted with the web application and intentionally triggered an error
- The application returned a **verbose stack trace** revealing the framework, version, file paths, and internal logic — a clear sign of debug mode being active in production
- This information could be used by an attacker to craft targeted exploits

**Key Finding:**  
The application was running in debug mode. The error output contained:
- Framework name and version
- File paths on the server
- Internal function call stack

**Remediation I noted:**  
Always set `DEBUG=False` in production. Never expose stack traces to end users. Use a generic error page instead.

---

### Task 3 — A03: Software Supply Chain Failures

**Concept:**  
Modern applications rely heavily on third-party libraries and components (npm packages, Python libraries, Docker images, APIs). If these components are:
- Outdated with known CVEs
- Compromised by a malicious actor
- Misconfigured in their integration

...the entire application is at risk regardless of how well the first-party code is written.

**What I Did:**
- Examined the application's dependency information
- Identified an outdated component being used
- Understood how a supply chain attack through a compromised package could introduce a backdoor
- Tested an API endpoint that accepted user input to manipulate how the application interacted with a downstream component

**Key Observation:**  
The application used `user-supplied input` to construct requests to a third-party service without validation — allowing manipulation of the downstream system's behavior.

---

### Task 4 — A06: Insecure Design

**Concept:**  
Insecure design means the application lacks the security controls it *should have had from the start*. Unlike misconfigurations (where the control exists but is set wrong), insecure design means the control was never built in.

Examples include:
- No rate limiting on sensitive actions
- Business logic flaws (e.g., allowing negative quantities in a shopping cart)
- Missing ownership checks
- Password reset flows that can be abused

**What I Did:**
- Explored the application's API endpoints
- Found an endpoint that accepted a `user_id` parameter directly — no authentication check was performed before returning that user's data
- Confirmed the flaw by modifying the user ID to access another user's profile
- The API returned full user details, confirming the design never considered authorization

**Key Observation:**  
This wasn't a code bug — the application was *designed* to accept any user ID without verification. The fix requires architectural changes, not just a patch.

---

### Task 5 — A10: Server-Side Request Forgery (SSRF)

**Concept:**  
SSRF occurs when an attacker can make the server send HTTP requests to an unintended target — often internal services not exposed to the internet. 

Common targets:
- Cloud metadata services (e.g., `http://169.254.169.254/` on AWS)
- Internal admin panels
- Internal databases
- Other microservices

**What I Did:**
- Found a feature in the application that accepted a URL as input (e.g., a "fetch preview" or "import from URL" functionality)
- Modified the URL to point to `http://127.0.0.1/admin` (localhost)
- The server fetched the internal admin panel and returned its content to me
- This demonstrated that an external attacker can read internal-only resources by weaponizing the server itself

**Payload used:**
```
http://127.0.0.1/admin
http://localhost/internal-api
```

**Key Observation:**  
The server had no URL validation or allowlist — it blindly fetched whatever URL was provided. Internal services trusted requests from `127.0.0.1` as safe, creating the perfect attack chain.

---

## 💡 What I Learned

### Technical Takeaways

| Vulnerability | What I Learned |
|---------------|---------------|
| **Security Misconfiguration (A02)** | Always harden production configs. Debug mode, default creds, and verbose errors are low-hanging fruit for attackers. Use security checklists before deploying. |
| **Supply Chain Failures (A03)** | Regularly audit dependencies with tools like `npm audit`, `pip-audit`, or `Snyk`. Pin versions and review changelogs before upgrading. |
| **Insecure Design (A06)** | Security must be designed in from the start. Threat modeling during the design phase prevents flaws that are expensive to fix post-deployment. |
| **SSRF (A10)** | Never allow user-supplied URLs to be fetched server-side without a strict allowlist. Block access to `169.254.x.x`, `127.0.0.1`, and internal IP ranges. |

### Conceptual Takeaways

- **Design flaws ≠ code bugs.** A security misconfiguration can be fixed with a config change. A design flaw requires rethinking the system.
- **Trust boundaries matter.** SSRF exploits the implicit trust that internal services place on requests from localhost — breaking that assumption has massive impact.
- **Supply chain is an emerging battleground.** The SolarWinds and Log4Shell incidents showed that a single compromised dependency can affect thousands of organizations.
- **Threat modeling** (thinking like an attacker during the design phase) is one of the most effective ways to prevent this entire class of vulnerabilities.

### Tools/Concepts Encountered
- HTTP error analysis and stack trace reading
- API parameter manipulation (IDOR/design flaw)
- SSRF payload construction
- Supply chain dependency analysis concepts
- URL allowlisting as a defense

---
