# Phase 19: Automated Offensive Security Testing 🔴

> **Objective:** Replace manual penetration testing with automated AI-driven offensive security tools.
> The AI agent installs, configures, runs, and analyzes results from real security testing tools
> that actively attack the backend/API/infrastructure — not just read its code. This is the DYNAMIC
> testing layer that complements the static analysis performed in Phases 1-18.

---

## ⚠️ IMPORTANT: This Phase is NON-NEGOTIABLE

> This phase uses **real security tools** that perform **actual attacks** against your backend.
> It is NOT static analysis — it is dynamic offensive testing.
> **Do NOT skip this phase.** Do NOT claim "tools are unavailable" without attempting installation first.

---

## 🚨 ANTI-CHEATING RULES FOR THIS PHASE

### Rule OT-A: MANDATORY `run_command` PROOF
> Every tool execution MUST be performed via an actual `run_command` / terminal tool call.
> You MUST show the **real terminal output** — not text you wrote from memory.
> **Writing fake tool output = CHEATING.** If you cannot run the command, mark as MANUAL_CHECK.
> The user can verify by checking the tool call history in the conversation.

### Rule OT-B: MAXIMUM 2 MANUAL_CHECK ALLOWED
> You may mark **at most 2 checks** as MANUAL_CHECK due to tool unavailability.
> If 3 or more tools fail to install, you MUST troubleshoot or try alternative installation
> methods (pip, npm, brew, choco, scoop, docker, direct download).
> **Claiming all tools are unavailable = PHASE INVALID. Start over.**

### Rule OT-C: HACKER MINDSET R6 ON ALL FIXES
> If you fix any finding in this phase, you MUST apply Rule R6 (Hacker Mindset Verification):
> 1. Can the fix be bypassed?
> 2. Does the fix create a new vulnerability?
> 3. Would a penetration tester find this fix adequate?
> **No R6 verification = fix is NOT accepted.**

---

## 🛠️ TOOL INSTALLATION

Before running any tests, install the required tools:

```bash
# Nuclei - Fast vulnerability scanner
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
# Or: brew install nuclei / choco install nuclei / docker pull projectdiscovery/nuclei

# sqlmap - Automated SQL Injection
pip install sqlmap
# Or: apt install sqlmap / brew install sqlmap

# Trivy - Container & Dependency Scanner
brew install trivy  # macOS
apt install trivy   # Debian/Ubuntu
choco install trivy # Windows

# testssl.sh - SSL/TLS Testing
git clone https://github.com/drwetter/testssl.sh.git

# ffuf - Web Fuzzer
go install github.com/ffuf/ffuf/v2@latest
# Or: brew install ffuf / apt install ffuf

# hey - HTTP Load Testing
go install github.com/rakyll/hey@latest

# kube-hunter - Kubernetes Penetration Testing
pip install kube-hunter

# ScoutSuite - Cloud Security Auditing
pip install scoutsuite
```

---

## 📋 PHASE 19 CHECKS

### [OT1] API Vulnerability Scan (Nuclei)
```
Action: Run Nuclei against all API endpoints
Commands:
  # Create target list from API routes
  nuclei -u https://[api-base-url] -t http/vulnerabilities/ -t http/cves/ -t http/exposures/
  nuclei -u https://[api-base-url] -t http/misconfiguration/ -t http/default-logins/
Analyze:
  - Known CVEs in the stack
  - Exposed debug/admin endpoints
  - Default credentials on services
  - Information disclosure
  - Misconfigured CORS/headers
  - Exposed .env / .git / swagger endpoints
Minimum citations: 8 findings from Nuclei output
```

### [OT2] SQL/NoSQL Injection Testing (sqlmap)
```
Action: Test all input endpoints for injection vulnerabilities
Commands:
  # Test specific endpoint
  sqlmap -u "https://[api-url]/endpoint?param=value" --batch --level=3 --risk=2
  # Test POST endpoints
  sqlmap -u "https://[api-url]/endpoint" --data="param=value" --batch
  # Test with authentication
  sqlmap -u "https://[api-url]/endpoint" --cookie="session=TOKEN" --batch
Analyze:
  - SQL injection points found
  - NoSQL injection possibilities
  - Blind injection vulnerabilities
  - Time-based injection
  - Database type and version exposed
Minimum citations: 5 findings
```

### [OT3] Container Security Scan (Trivy)
```
Action: Scan all Docker images and Dockerfiles
Commands:
  trivy image [image-name:tag]
  trivy fs --security-checks vuln,config /path/to/project
  trivy config /path/to/dockerfile
  trivy config /path/to/kubernetes/manifests/
Analyze:
  - OS package vulnerabilities in containers
  - Application dependency CVEs
  - Dockerfile misconfigurations (running as root, etc.)
  - Kubernetes manifest misconfigurations
  - Secrets in container layers
Minimum citations: 6 findings
```

### [OT4] SSL/TLS Configuration Testing
```
Action: Test SSL/TLS configuration of all exposed endpoints
Commands:
  ./testssl.sh https://[api-url]
  nmap --script ssl-enum-ciphers -p 443 [host]
Analyze:
  - Weak TLS versions (reject 1.0/1.1)
  - Weak cipher suites
  - Certificate chain issues
  - HSTS configuration
  - OCSP stapling
  - Key exchange strength
Minimum citations: 4 findings
```

### [OT5] API Fuzzing (ffuf)
```
Action: Fuzz API endpoints for hidden paths and parameter injection
Commands:
  # Directory/endpoint fuzzing
  ffuf -u https://[api-url]/FUZZ -w /path/to/wordlist.txt -mc 200,301,302,403
  # Parameter fuzzing
  ffuf -u https://[api-url]/endpoint?FUZZ=value -w /path/to/params.txt
  # IDOR testing
  ffuf -u https://[api-url]/users/FUZZ -w <(seq 1 1000) -mc 200
Analyze:
  - Hidden administrative endpoints
  - IDOR vulnerabilities (accessing other users' data)
  - Parameter injection points
  - Rate limiting bypass
  - Authentication bypass paths
Minimum citations: 4 findings
```

### [OT6] Load & Stress Testing (hey)
```
Action: Test API resilience under load
Commands:
  # Basic load test (200 requests, 50 concurrent)
  hey -n 200 -c 50 https://[api-url]/endpoint
  # Sustained load test
  hey -z 30s -c 100 https://[api-url]/endpoint
  # With authentication
  hey -n 200 -c 50 -H "Authorization: Bearer TOKEN" https://[api-url]/endpoint
Analyze:
  - Response time degradation under load
  - Error rate increase
  - Memory/CPU spikes (check via monitoring)
  - Connection pool exhaustion
  - Rate limiting effectiveness
Minimum citations: 3 findings
```

### [OT7] Cloud Security Audit (if applicable)
```
Action: Audit cloud infrastructure security
Commands:
  # AWS
  scout aws --report-dir ./scout-report
  # GCP
  scout gcp --report-dir ./scout-report
  # Azure
  scout azure --report-dir ./scout-report
  # Kubernetes
  kube-hunter --remote [cluster-ip]
  kube-hunter --pod  # from within a pod
Analyze:
  - Public S3 buckets / Storage blobs
  - Overly permissive IAM roles
  - Unencrypted databases
  - Security groups allowing 0.0.0.0/0
  - Kubernetes RBAC misconfigurations
  - Missing audit logging
Minimum citations: 5 findings (if cloud infra exists, otherwise mark N/A)
```

### [OT8] Dependency Vulnerability Scan
```
Action: Deep scan all backend dependencies
Commands:
  # Node.js
  npm audit --json
  npx snyk test
  # Python
  pip-audit
  safety check
  # Go
  govulncheck ./...
  # Java
  ./mvnw dependency-check:check
  # Ruby
  bundle-audit check
  # .NET
  dotnet list package --vulnerable
Analyze:
  - Critical CVEs (CVSS >= 9.0)
  - High CVEs (CVSS >= 7.0)  
  - Known exploited vulnerabilities (KEV)
  - Outdated framework versions
Minimum citations: 5 findings
```

---

## 🔧 REMEDIATION WITHIN THIS PHASE

Unlike Phases 1-18, you MAY fix Critical and High findings discovered in this phase immediately.
For each fix:
1. Show the tool output that identified the vulnerability
2. Apply the fix
3. Re-run the specific tool to verify the fix worked
4. Document before/after results

---

## ⚠️ TOOL UNAVAILABILITY PROTOCOL

If a tool genuinely cannot be installed:
1. **Document exactly what you tried** and the error message
2. **Perform manual equivalent checks** using curl, grep, and log analysis
3. **Mark the check as PARTIAL** in the gate report
4. **Never mark as PASS** if the tool couldn't run — mark as MANUAL_CHECK

---

## 📊 PHASE 19 GATE REPORT

```
╔══════════════════════════════════════════════════════╗
║  PHASE 19 GATE — Automated Offensive Testing         ║
╠══════════════════════════════════════════════════════╣
║ [OT1] Nuclei API Scan:       [PASS/FAIL/PARTIAL]    ║
║ [OT2] SQL Injection:         [PASS/FAIL/PARTIAL]    ║
║ [OT3] Container Security:    [PASS/FAIL/PARTIAL]    ║
║ [OT4] SSL/TLS Config:        [PASS/FAIL/PARTIAL]    ║
║ [OT5] API Fuzzing:           [PASS/FAIL/PARTIAL]    ║
║ [OT6] Load Testing:          [PASS/FAIL/PARTIAL]    ║
║ [OT7] Cloud Security:        [PASS/FAIL/PARTIAL/NA] ║
║ [OT8] Dependency Vulns:      [PASS/FAIL/PARTIAL]    ║
╠══════════════════════════════════════════════════════╣
║ Tools Successfully Run:      [N/8]                   ║
║ Critical Findings:           [N]                     ║
║ Findings Fixed:              [N]                     ║
║ Remaining Risks:             [N]                     ║
╠══════════════════════════════════════════════════════╣
║ VERDICT: [PROCEED TO PHASE 20 / BLOCK — FIX FIRST]  ║
╚══════════════════════════════════════════════════════╝
```

> ⛔ **STOP — Do NOT proceed to Phase 20 until the user confirms.**

---

## 📌 MINIMUM CITATION REQUIREMENTS

| Check | Minimum Citations |
|-------|-------------------|
| OT1 - Nuclei | 8 |
| OT2 - SQLi | 5 |
| OT3 - Container | 6 |
| OT4 - SSL/TLS | 4 |
| OT5 - Fuzzing | 4 |
| OT6 - Load Test | 3 |
| OT7 - Cloud | 5 |
| OT8 - Dependencies | 5 |
| **Total** | **40** |

> Every citation MUST include: tool name, exact output snippet, endpoint/service affected, and severity.
