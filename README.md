# ☁️ Backend & Cloud Auditor Skill

[![Environments](https://img.shields.io/badge/Environments-Docker%20%7C%20K8s%20%7C%20AWS%20%7C%20GCP-blue)](SKILL.md)

---

## ✨ What This Skill Does

When installed, your AI agent becomes an **Ethical Hacker (Red Team), Cloud Architect, and Security Auditor** capable of:

- 🏗️ **Architecture & Topology** — Monolith vs Microservices, Bounded Contexts, DI
- 🧠 **Business Logic & Integrity** — State machines, currency/float math, edge cases
- 🔌 **API & Omnichannel Gateway** — REST, GraphQL depth attacks, pagination, versioning
- ⚡ **Real-time & Async Systems** — WebSockets, Webhooks (HMAC), Server-Sent Events
- 🗄️ **Database & Data Architecture** — Sharding, replication, ACID, Saga pattern
- 💨 **Distributed Caching & Edge** — Redis, cache invalidation, stampede prevention
- 🔄 **Concurrency & Load** — Thread blocking, deadlocks, horizontal scaling
- 🔑 **Auth & Identity** — RBAC, multi-tenancy, JWT rotation, OAuth2
- 🛡️ **Advanced Security** — OWASP API Top 10, SSRF, BOLA, BFLA *(mandatory web search)*
- 💪 **Resilience & Fault Tolerance** — Chaos engineering, circuit breakers, bulkheads
- 📊 **Observability & Auditing** — Distributed tracing, ELK, SIEM, PII masking
- 🐳 **Infrastructure, Cloud & CI/CD** — Docker/K8s, secrets, CVEs *(mandatory web search)*
- 📜 **Compliance & DR** — GDPR, SOC2, disaster recovery *(mandatory web search)*
- 🔒 **Zero Trust & Service Mesh** — mTLS, SPIFFE, Istio, egress control
- 📦 **Supply Chain & GitOps** — SBOM, SLSA, ArgoCD, Terraform state
- 🕵️ **Advanced Threats & Forensics** — KMS, egress, WORM, serverless, insider threats
- ✅ **Final Delivery + Fresh-Eyes Re-Analysis** — Executive summary, matrices, mandatory second pass
- 🔧 **Structured Remediation (Phase 18)** — Sprint-based fixes with Hacker Mindset R6 verification
- 🔴 **Automated Offensive Testing (Phase 19)** — AI runs real security tools (Nuclei, sqlmap, Trivy, ffuf, ScoutSuite) with mandatory database freshness and anti-cheating enforcement
- 🛡️ **Pre-Deployment Sentry Validation (Phase 20)** — Real-traffic testing, attack simulation, deployment verdict

---

## 🚀 Install in 30 Seconds

```bash
npx skills add https://github.com/abosalehworld-oss/backend-cloud-auditor-skill
```

Or globally:

```bash
npx skills add https://github.com/abosalehworld-oss/backend-cloud-auditor-skill -g -y
```

---

## 🎯 Supported Environments & Frameworks

| Environment | Supported |
|-------------|-----------|
| **Localhost / VPS** | ✅ |
| **Docker / Docker Compose** | ✅ |
| **Kubernetes** | ✅ |
| **Serverless (Lambda, Cloud Functions)** | ✅ |
| **AWS / GCP / Azure** | ✅ |

| Framework | Detection |
|-----------|-----------|
| **Node.js / Express** | `package.json` + express, `app.js/ts` |
| **Python / Django** | `manage.py`, `settings.py` |
| **Python / FastAPI** | `main.py` with FastAPI import |
| **Go** | `go.mod`, `main.go` |
| **Java / Spring Boot** | `pom.xml` with spring-boot, `Application.java` |
| **PHP / Laravel** | `artisan`, `routes/api.php` |
| **Ruby / Rails** | `Gemfile` with rails, `routes.rb` |
| **.NET / ASP.NET** | `*.csproj` with AspNetCore, `Program.cs` |

---

## 🛡️ Anti-Laziness Enforcement System (10 Layers)

This skill is engineered to **force thorough review** even from AI agents that tend to skip steps.

### 1. Proof-of-Work Citations (193 Minimum)
Every finding MUST include exact file path, line numbers, and copied code snippet.

### 2. Mandatory Phase Gates (20 Gates)
Each of the 20 phases has a structured checklist that must be completed with evidence before the AI can proceed.

### 3. Mandatory STOP Points
After each phase, the AI MUST output a structured report and wait for acknowledgment.

### 4. Cross-Reference Verification Matrix
The final phase requires a matrix proving every file was examined — if any phase shows 0 files, the review is invalid.

### 5. Fresh-Eyes Re-Analysis (4-Layer Reminder)
After all analysis phases, the AI performs a second independent pass — proven to catch Critical bugs missed on first pass.

### 6. Hacker Mindset R6
After every fix, the AI verifies from an attacker's perspective that no new vulnerabilities were introduced.

### 7. Anti-Premature-Celebration
AI is blocked from declaring "done" after remediation — offensive testing and Sentry phases still remain.

### 8. 🆕 Mandatory `run_command` Proof (Phase 19)
Every security tool execution MUST be via real terminal commands — fabricating tool output = cheating.

### 9. 🆕 Maximum 2 MANUAL_CHECK (Phase 19)
AI cannot claim all tools are unavailable — at most 2 checks can be manual.

### 10. 🆕 Mandatory Database Freshness (Phase 19)
AI MUST update vulnerability databases before scanning + web search for latest tool versions.

---

## 📂 Skill Structure

```
backend-cloud-auditor-skill/
├── SKILL.md                              ← Main entry + enforcement rules
├── 01-architecture-topology.md           ← Phase 1: Monolith/Microservices
├── 02-business-logic-integrity.md        ← Phase 2: State machines, math
├── 03-api-omnichannel-gateway.md         ← Phase 3: REST, GraphQL
├── 04-realtime-async-systems.md          ← Phase 4: WebSockets, webhooks
├── 05-database-data-architecture.md      ← Phase 5: Sharding, ACID, Saga
├── 06-distributed-caching-edge.md        ← Phase 6: Redis, invalidation
├── 07-concurrency-load-scalability.md    ← Phase 7: Threading, scaling
├── 08-auth-identity-management.md        ← Phase 8: RBAC, JWT, OAuth2
├── 09-advanced-security-defenses.md      ← Phase 9: OWASP API, SSRF
├── 10-resilience-fault-tolerance.md      ← Phase 10: Chaos, circuit breakers
├── 11-observability-auditability.md      ← Phase 11: Tracing, SIEM
├── 12-infrastructure-cloud-cicd.md       ← Phase 12: Docker, K8s, secrets
├── 13-compliance-privacy-dr.md           ← Phase 13: GDPR, SOC2, DR
├── 14-zero-trust-service-mesh.md         ← Phase 14: mTLS, SPIFFE
├── 15-supply-chain-gitops.md             ← Phase 15: SBOM, SLSA
├── 16-advanced-threats-forensics.md      ← Phase 16: APT, insider threats
├── 17-final-delivery-readiness.md        ← Phase 17: Summary + Fresh-Eyes
├── 18-structured-remediation.md          ← Phase 18: Sprint fixes + R6
├── 19-automated-offensive-testing.md     ← Phase 19: Nuclei, sqlmap, Trivy, ffuf, hey
└── 20-pre-deployment-sentry-validation.md ← Phase 20: Sentry + traffic testing
```

---

## 💬 How to Use with Your AI Agent

### Start a Full Review
```
Review my Node.js/Express API using the backend-cloud-auditor skill.
Start from Phase 1 and follow ALL mandatory gates. Do not skip phases.
My project is at: [path to your project]
```

### Security-Only Review
```
Using the backend-cloud-auditor skill, run ONLY Phase 9 (Advanced Security)
on my Django API. Check for OWASP API Top 10 and current-year CVEs.
```

### Infrastructure Check
```
Using the backend-cloud-auditor skill, run Phase 12 (Infrastructure)
on my Kubernetes deployment. Check for exposed secrets and misconfigured RBAC.
```

---

## 🤝 Contributing

Found a common vulnerability pattern? Have a framework-specific check to add?  
PRs are welcome!

---

## 📜 License

MIT — free to use, share, and modify.

---

## 🌟 Activate on skills.sh

```bash
npx skills add https://github.com/abosalehworld-oss/backend-cloud-auditor-skill -g -y
```

> Built with ❤️ for backend engineers who take security seriously.

---

## 👤 Author

**Mohamed Saleh** 🇪🇬  
Egypt — Mobile & Software Developer  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Mohamed%20Saleh-blue?logo=linkedin)](https://www.linkedin.com/in/mr-mohamed-saleh/)
