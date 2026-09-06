# Phase 16: Advanced Threats & Forensics 🕵️‍♂️

> ⚡ **REMINDER:** After completing this phase, Phase 17 requires a **MANDATORY FRESH-EYES
> RE-ANALYSIS** where you must re-read ALL critical files as a different AI agent.
> Do NOT forget this when you reach Phase 17. It is the MOST IMPORTANT step.

> **Objective:** Evaluate the system against Advanced Persistent Threats (APTs), insider threats, data exfiltration, and forensic readiness. This phase assumes the perimeter IS breached and tests internal containment and post-breach observability.

---

## 📋 ADVANCED THREAT CHECKS

### CHECK APT-1: Cryptographic Standards & Key Management
```
ACTION: Use `search_web` to search for "NIST current recommended cryptographic algorithms [Current Year]".
WHAT TO CHECK:
  ❑ Did you perform the web search for current NIST standards?
  ❑ Are the databases/S3 buckets using the algorithms retrieved from the search? (e.g., AES-256-GCM).
  ❑ Is a Key Management Service (AWS KMS, HashiCorp Vault) explicitly used instead of environment variables?

WHY IT MATTERS:
  Hardcoded keys or weak legacy algorithms (like MD5/SHA1 for hashing, or AES-ECB) allow attackers to decrypt stolen databases easily.

CITATION REQUIRED: Exact path to Terraform/Infra/Config file defining the DB encryption or KMS key.
SEVERITY: 🔴 CRITICAL — Complete data exposure.
```

### CHECK APT-2: Forensics Readiness & Immutable Logs (WORM)
```
WHAT TO CHECK:
  ❑ Are critical audit logs stored in an immutable state? (e.g., AWS S3 Object Lock, Append-Only ledgers).
  ❑ If an attacker gains `admin` access to the API, can they delete the logs via the codebase?

COMMON BUGS:
  🔴 The application uses standard `console.log` or writes to a local `.log` file that is deleted when the container restarts, destroying forensic evidence.

CITATION REQUIRED: Find the exact logger implementation file and trace the storage destination.
SEVERITY: 🟠 HIGH — Prevents post-breach investigation.
```

### CHECK APT-3: Egress Filtering & Data Exfiltration
```
WHAT TO CHECK:
  ❑ Can the backend servers make outbound requests to ANY IP address? (0.0.0.0/0).
  ❑ Are egress NetworkPolicies (K8s) or Security Groups configured to whitelist ONLY required third-party APIs (e.g., Stripe, Twilio)?

COMMON BUGS:
  🔴 An attacker exploiting an SSRF or RCE vulnerability forces the server to download malware or exfiltrate the DB to an external IP, because outbound traffic is completely unrestricted.

SEARCH PATTERNS:
  - `egress` in Kubernetes NetworkPolicies or AWS Security Groups.
  - `cidr_blocks = ["0.0.0.0/0"]` in outbound rules.

CITATION REQUIRED: Show the exact network configuration file blocking outbound traffic.
SEVERITY: 🔴 CRITICAL — Enables lateral movement and data exfiltration.
```

### CHECK APT-4: Insider Threat & Two-Man Rule
```
WHAT TO CHECK:
  ❑ For high-risk endpoints (e.g., `/api/admin/drop-tenant`, `/api/refunds`), is there a multi-party authorization requirement or step-up MFA?
  ❑ Can a single compromised admin token execute destructive actions without secondary verification?

WHY IT MATTERS:
  A compromised developer account or a malicious insider can wipe out databases if no "Two-Man Rule" or MFA is enforced on destructive endpoints.

SEVERITY: 🟠 HIGH — Insider destruction.
```

### CHECK APT-5: Serverless IAM & Cold Start Poisoning
```
WHAT TO CHECK:
  ❑ Does each serverless function (Lambda/CloudFunction) have a highly restrictive, single-purpose IAM role?

COMMON BUGS:
  🔴 Multiple Lambda functions share a single `AdministratorAccess` policy or `Action: "*"` role. If one function is exploited, the attacker owns the entire AWS account.

SEARCH PATTERNS:
  - `iamRoleStatements` in Serverless Framework.
  - `Role: arn:aws:iam::...` in Terraform/CloudFormation.

CITATION REQUIRED: Path to `serverless.yml` or IAM definition.
SEVERITY: 🔴 CRITICAL — Privilege escalation.
```

### CHECK APT-6: Advanced WAF & Bot Management
```
ACTION: Use `search_web` for "Latest API Bot bypass techniques and L7 DDoS vectors [Current Year]".
WHAT TO CHECK:
  ❑ Are there WAF rules explicitly defined in the infra code based on current-year threats?
  ❑ Is there adaptive Rate Limiting implemented (not just basic IP-based which fails against distributed proxies)?

WHY IT MATTERS:
  Without L7 protection, automated botnets can scrape all user data, brute-force logins, or cause an Application-layer DDoS that caching alone cannot stop.

SEVERITY: 🟠 HIGH — Application exhaustion and scraping.
```
