# Phase 12: Infrastructure, Cloud & CI/CD Security ☁️

> **Objective:** Evaluate the complete deployment lifecycle, from containerization (Docker) and orchestration (Kubernetes) to Cloud Provider IAM and CI/CD pipeline security. This phase ensures that the application is not compromised from the outside in, regardless of how secure the application code itself is.

---

## ⚠️ THIS IS A HIGH-STAKES PHASE
> Infrastructure misconfigurations (like a public S3 bucket or a root Docker container) lead to instantaneous, automated exploitation. **You must act as a DevSecOps Engineer.**

---

## 🌐 MANDATORY WEB SEARCH (CVE DETECTION)

You cannot audit infrastructure dependencies using static, outdated training data. You **MUST** execute the following workflow:

### Step 1: Framework & OS Identification
Identify the base OS (e.g., Alpine 3.14, Ubuntu 20.04) and core infrastructure tools (e.g., Nginx 1.18, Redis 5.0) used in the Dockerfiles or deployment scripts.

### Step 2: Mandatory `search_web` Execution
Use your `search_web` tool to search:
`"CVE [Technology] [Version] [CURRENT_YEAR] exploit"`
*(e.g., `"CVE Nginx 1.18 2024 exploit" or `"CVE Alpine 3.14 vulnerabilities"`)*

### Step 3: Vulnerability Mapping
List any High or Critical CVEs found on the live web that apply to the infrastructure files. If you skip this search, your audit is **INVALID**.

---

## 📋 I1: DOCKER & CONTAINERIZATION SECURITY

### 1.1 The Root User Anti-Pattern
```
WHAT TO CHECK:
  ❑ Is the Dockerfile running as `root` (the default)?
  ❑ Is there a `USER` directive switching to a non-root user (e.g., `USER node`, `USER appuser`) before the `CMD` or `ENTRYPOINT`?

WHY IT MATTERS:
  If a container runs as root and an attacker achieves Remote Code Execution (RCE) via a vulnerability in the app (like a prototype pollution or insecure deserialization), they have ROOT access inside the container. If there's a container breakout vulnerability, they gain ROOT on the host node.

SEVERITY: 🔴 CRITICAL
CITATION REQUIRED: Copy the end of the Dockerfile showing the USER context.
```

### 1.2 Base Image Hygiene & Pinning
```
WHAT TO CHECK:
  ❑ Are images using the `:latest` tag? (This breaks deterministic builds).
  ❑ Are heavy, full OS images used instead of slim/alpine variants? (e.g., `FROM ubuntu` instead of `FROM alpine:3.19`).
  ❑ Are images pulled from untrusted registries?
  ❑ Are image hashes (SHA256) used for absolute immutability? (`FROM node@sha256:12345...`).
```

### 1.3 Multi-Stage Builds & Secrets in Layers
```
WHAT TO CHECK:
  ❑ Are multi-stage builds used to separate the build environment (compilers, SDKs) from the runtime environment?
  ❑ Are SSH keys, NPM tokens, or AWS credentials passed via `ENV` or `ARG`? (These are permanently baked into the image history!)
  ❑ Should they use Docker BuildKit's `--mount=type=secret` instead?

ANTI-PATTERN EXAMPLE:
  🔴 `ENV AWS_ACCESS_KEY_ID=AKIA...` (Anyone pulling the image can run `docker history` and steal the key).
```

### 1.4 Container Capabilities & Seccomp
```
WHAT TO CHECK (in docker-compose.yml or run scripts):
  ❑ Is `--privileged` used? (NEVER ALLOW THIS IN PRODUCTION).
  ❑ Are Linux capabilities dropped? (`--cap-drop=ALL`).
  ❑ Is the root filesystem mounted as read-only? (`read_only: true`).
```

---

## 📋 I2: KUBERNETES (K8s) ORCHESTRATION SECURITY

### 2.1 Role-Based Access Control (RBAC) & Service Accounts
```
WHAT TO CHECK (in .yaml manifests):
  ❑ Are default Service Accounts used? (They automatically mount API tokens into every pod).
  ❑ `automountServiceAccountToken: false` must be set unless the pod explicitly needs to talk to the K8s API.
  ❑ Are ClusterRoles granting `*` (wildcard) permissions?

ANTI-PATTERN EXAMPLE:
  🔴 A web pod gets compromised. Because `automountServiceAccountToken` was true, the attacker reads `/var/run/secrets/kubernetes.io/serviceaccount/token` and uses it to take over the entire cluster.
```

### 2.2 Network Policies (Zero Trust)
```
WHAT TO CHECK:
  ❑ Is there a Default Deny NetworkPolicy?
  ❑ Can the `frontend` pod talk directly to the `database` pod? (It shouldn't, only the `backend` should).

WHY IT MATTERS:
  By default, Kubernetes allows ANY pod to talk to ANY other pod. If the frontend is compromised, the attacker can scan the internal network and brute-force the database.
```

### 2.3 Resource Limits & Quotas
```
WHAT TO CHECK:
  ❑ Does every Deployment/Pod have `resources.limits` and `resources.requests` defined for CPU and Memory?

SEVERITY: 🟠 HIGH — Without limits, a memory leak in one pod will consume the entire Node's RAM, causing the Kubelet to crash and evicting ALL other pods (Cascading Failure).
```

---

## 📋 I3: CLOUD PROVIDER & IAM SECURITY (AWS/GCP/AZURE)

### 3.1 Hardcoded Cloud Credentials
```
WHAT TO CHECK:
  ❑ Are `AWS_ACCESS_KEY_ID` or `GOOGLE_APPLICATION_CREDENTIALS` hardcoded in source code or committed to `.env` files?
  ❑ Are IAM Roles for Service Accounts (IRSA on AWS) or Workload Identity (GCP) used instead of long-lived access keys?

SEVERITY: 🔴 CRITICAL — Instant cloud takeover.
```

### 3.2 Public Storage Buckets
```
WHAT TO CHECK (in Terraform/CloudFormation or code):
  ❑ Are S3 buckets / GCP Cloud Storage buckets marked as `public-read` globally?
  ❑ Is sensitive data (DB backups, user uploads) stored in buckets without strict bucket policies?
```

### 3.3 SSRF Cloud Metadata Protection
```
WHAT TO CHECK (AWS Specific):
  ❑ Is IMDSv2 (Instance Metadata Service v2) enforced on EC2 instances?
  ❑ If IMDSv1 is active, an SSRF vulnerability in the app allows an attacker to fetch `http://169.254.169.254/latest/meta-data/iam/security-credentials/` and steal the server's IAM role keys.
```

---

## 📋 I4: CI/CD PIPELINE SECURITY

### 4.1 Pipeline Secrets & Injection
```
WHAT TO CHECK (in .github/workflows, .gitlab-ci.yml):
  ❑ Are secrets echoed in pipeline logs? (`echo $DB_PASSWORD > .env`).
  ❑ Are pipeline secrets scoped to specific environments (Production vs Staging)?
  ❑ Can a malicious Pull Request exfiltrate secrets? (e.g., executing arbitrary code during the CI build process).
```

### 4.2 Security Scanners (SAST/DAST/SCA)
```
WHAT TO CHECK:
  ❑ Is Static Application Security Testing (SAST) running on PRs? (e.g., SonarQube, CodeQL).
  ❑ Is Software Composition Analysis (SCA) running to check dependencies? (e.g., Dependabot, Snyk).
  ❑ Is container image scanning integrated into the pipeline before pushing to the registry? (e.g., Trivy, Clair).
```

### 4.3 Branch Protection & Code Owners
```
WHAT TO CHECK:
  ❑ Can a developer push directly to the `main` or `master` branch?
  ❑ Is a pull request required?
  ❑ Are at least 2 approving reviews required?
  ❑ Do changes to infrastructure files (`terraform/`, `k8s/`) require review from the DevOps/Security team (CODEOWNERS)?
```

---

## 🛡️ INFRASTRUCTURE SECURITY MATRIX

Use this matrix to grade the infrastructure maturity:

| Domain | ❌ Insecure (Fail) | 🟡 Acceptable (Warn) | ✅ Secure (Pass) |
|--------|-------------------|----------------------|------------------|
| **Docker User** | Runs as `root` | Non-root but no read-only FS | Non-root + `read_only: true` |
| **Secrets** | `.env` in Git | `.env` in CI variables | HashiCorp Vault / Secret Manager |
| **K8s Network** | Open communication | Namespaces separated | Strict Default Deny NetworkPolicy |
| **Cloud IAM** | Admin keys in code | IAM User with keys | Temporary Role Assumption (OIDC) |
| **CI/CD** | Direct push to main | PR required (1 review) | PR required + SAST + SCA scanning |

---

## 🚦 PHASE 12 GATE — MANDATORY CHECKLIST

You MUST complete ALL items before proceeding:

```
PHASE 12 GATE CHECKLIST:
  □ [I1.1-1.4] Docker root, base images, and capabilities verified
  □ [I2.1-2.3] Kubernetes RBAC, Network Policies, and Limits verified
  □ [I3.1-3.3] Cloud IAM, S3 visibility, and IMDSv2 verified
  □ [I4.1-4.3] CI/CD pipeline, SAST scanners, and Branch Protection verified
  □ 🌐 WEB SEARCH EXECUTED for CVEs on infrastructure dependencies
  □ Minimum 10 code citations provided (Dockerfiles, YAML, CI configs)
  □ Infrastructure Security Matrix evaluated
```

### Gate Report Format:

```
═══════════════════════════════════════════════════════
  ✅ PHASE 12 COMPLETE: Infrastructure, Cloud & CI/CD
  📊 Findings: [X] Critical | [Y] High | [Z] Medium | [W] Low
  📋 Gate Status: [PASSED/FAILED] ([checked]/7 items)

  ☁️ DevOps Health Score: [X/10]
  Docker Security: [Good/Warning/Critical]
  K8s/Cloud Security: [Good/Warning/Critical]
  CI/CD Maturity: [Good/Warning/Critical]

  🌐 Web Search Results (CVEs Found): [Count]
═══════════════════════════════════════════════════════
```

### ⛔ STOP POINT
**DO NOT proceed to Phase 13 until:**
1. All gate items are checked and justified.
2. The Web Search results for CVEs are presented.
3. User acknowledges the findings (Critical infrastructure bugs require immediate patching).
