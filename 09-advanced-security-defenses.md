# Phase 9: Advanced Security & Defenses 🛡️

> **Objective:** Detect cutting-edge vulnerabilities based on the most recent industry data. Hackers evolve daily; a static checklist is insufficient. This phase mandates active threat hunting.

## ⚠️ ANTI-LAZINESS & MANDATORY WEB SEARCH GATE

You MUST NOT rely on your training data for this phase. The threat landscape changes daily. 

**ACTION: STOP AND SEARCH.** 
Before examining any code, you MUST use the `search_web` tool to search for the EXACT CURRENT YEAR you are operating in.
Search Query 1: `"OWASP Top 10 API Security Risks <insert current year here>"`
Search Query 2: `"Latest Backend Zero Day vulnerabilities and exploit techniques <insert current year here>"`

**VERIFICATION:** If you do not output the exact search queries used and the top 3 newly discovered attack vectors for the current year in your report, this entire phase is marked as **FAILED**.

---

## 📋 ADVANCED THREAT CHECKS

### CHECK SEC-1: Broken Object Level Authorization (BOLA/IDOR)
```
WHAT TO CHECK:
  ❑ When an endpoint takes an ID (e.g., `/api/receipts/555`), does the backend verify that the currently authenticated user OWNS receipt 555?
  ❑ Are Predictable IDs (Auto-incrementing integers) used instead of UUIDv4 or KSUIDs?
  
SEARCH PATTERNS:
  - `SELECT * FROM receipts WHERE id = ?` (Vulnerable if `user_id` is not checked).
  - Look for endpoints updating/deleting resources based on path parameters.

COMMON BUGS:
  🔴 An attacker logs in, gets their JWT. They see their profile is `/api/users/100`. They change the URL to `/api/users/101` and view another user's PII because the server only validated the JWT signature, not the ownership of record 101.

SEVERITY: 🔴 CRITICAL — The #1 API vulnerability.
```

### CHECK SEC-2: Broken Function Level Authorization (BFLA)
```
WHAT TO CHECK:
  ❑ Can a standard user simply change the URL path from `POST /api/users/profile` to `POST /api/admin/users/delete` and succeed because the backend assumed the UI would hide the button?
  ❑ Are admin routes protected by a middleware/guard that re-verifies the role at the router level?

SEVERITY: 🔴 CRITICAL
```

### CHECK SEC-3: Server-Side Request Forgery (SSRF)
```
WHAT TO CHECK:
  ❑ Does the backend accept a URL from the user and fetch it? (e.g., fetching a profile picture, webhook targets, PDF generation from URL).
  ❑ Is the URL validated against a strict allowlist?
  ❑ Can the backend be tricked into querying internal network addresses?

CLOUD METADATA ATTACK (IMDS):
  🔴 AWS EC2 instance metadata is accessible at `http://169.254.169.254/`. If the backend uses `axios.get(req.body.url)`, the attacker passes the metadata IP and steals the server's IAM temporary credentials.
  ✅ Fix: Enforce IMDSv2 (requires headers) and block internal IPs (`10.0.0.0/8`, `127.0.0.0/8`).

SEVERITY: 🔴 CRITICAL — Complete cloud infrastructure takeover.
```

### CHECK SEC-4: Mass Assignment & BOPLA
```
WHAT TO CHECK:
  ❑ Does the API blindly merge the user's JSON payload into the database record?
  ❑ Can a user send `{"is_admin": true, "account_balance": 999999}` and have it saved?
  
REQUIRED FIX:
  Must use Data Transfer Objects (DTOs), strict validation schemas (Zod, Pydantic, Joi), or explicit field selection (e.g., `user.update({ name: req.body.name })`).
```

### CHECK SEC-5: Insecure Deserialization & Prototype Pollution
```
WHAT TO CHECK:
  ❑ Python/Java: Is `pickle.loads()` or `ObjectInputStream.readObject()` used on user input? (Instant RCE).
  ❑ Node.js: Are objects deeply merged without checking for `__proto__` or `constructor.prototype`?

COMMON BUGS (Prototype Pollution in JS):
  🔴 `lodash.merge({}, JSON.parse('{"__proto__": {"isAdmin": true}}'))` pollutes the global Object prototype, making every object in the app an admin.
```

---

## 🚦 PHASE 9 GATE — MANDATORY CHECKLIST

```
PHASE 9 GATE CHECKLIST:
  □ 🌐 WEB SEARCH EXECUTED for current year OWASP / Vulnerabilities
  □ [SEC-1] BOLA and Predictable IDs checked on all ID-based routes
  □ [SEC-2] BFLA admin route protection verified
  □ [SEC-3] SSRF protections, internal IP blocking verified
  □ [SEC-4] Mass Assignment (BOPLA) and DTOs verified
  □ [SEC-5] Deserialization and Prototype Pollution checked
  □ Minimum 8 code citations provided
```

### OWASP API Compliance Matrix:
```
┌──────────────────────────────────────┬─────────┬──────────┐
│ Modern OWASP Category (Latest Version) │ Status  │ Findings │
├──────────────────────────────────────┼─────────┼──────────┤
│ BOLA (Broken Object Level Auth)      │ ✅/⚠️/❌ │ [count]  │
│ BFLA (Broken Function Level Auth)    │ ✅/⚠️/❌ │ [count]  │
│ BOPLA (Broken Object Property Auth)  │ ✅/⚠️/❌ │ [count]  │
│ SSRF (Server-Side Request Forgery)   │ ✅/⚠️/❌ │ [count]  │
│ Insecure Deserialization             │ ✅/⚠️/❌ │ [count]  │
└──────────────────────────────────────┴─────────┴──────────┘
```

### Gate Report Format:
```
═══════════════════════════════════════════════════════
  ✅ PHASE 9 COMPLETE: Advanced Security & Defenses
  📊 Findings: [X] Critical | [Y] High | [Z] Medium | [W] Low
  📋 Gate Status: [PASSED/FAILED] ([checked]/7 items)
  
  🌐 Web Search Results (New CVEs/OWASP updates): [Summarize]
═══════════════════════════════════════════════════════
```
### ⛔ STOP POINT: Wait for user acknowledgment.
