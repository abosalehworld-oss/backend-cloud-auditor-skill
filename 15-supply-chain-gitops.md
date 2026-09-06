# Phase 17: Supply Chain Security & GitOps 📦

> **Objective:** Evaluate the integrity of the software delivery pipeline. SolarWinds and Log4j proved that attackers don't need to hack your application; they just need to hack your dependencies or your deployment pipeline. This phase audits SLSA compliance, SBOM generation, and GitOps maturity.

---

## 📋 S1: SOFTWARE SUPPLY CHAIN (SLSA & SBOM)

### 1.1 Software Bill of Materials (SBOM)
```
WHAT TO CHECK:
  ❑ Is the CI/CD pipeline generating an SBOM (e.g., using Syft, Trivy, or SPDX formats) for every container image?
  ❑ Is the SBOM stored alongside the image in the container registry?
  
WHY IT MATTERS:
  When a new zero-day like Log4j drops, if you don't have an SBOM, you have to manually scan thousands of repositories to find out if you are vulnerable. An SBOM tells you instantly.
```

### 1.2 Artifact Signing & Provenance (Cosign / Sigstore)
```
WHAT TO CHECK:
  ❑ Are Docker images cryptographically signed after they are built? (e.g., using Cosign / Sigstore).
  ❑ Does the Kubernetes cluster have an Admission Controller (like Kyverno or OPA Gatekeeper) that REJECTS images if they don't have a valid cryptographic signature from the CI pipeline?

ANTI-PATTERN EXAMPLE:
  🔴 An attacker gains access to the Docker Registry and overwrites `backend:v1.0` with a malicious image. Kubernetes pulls and runs it.
  ✅ Fix: Kubernetes verifies the Cosign signature. The attacker doesn't have the CI pipeline's private key, so the signature is invalid, and Kubernetes refuses to run the malware.

SEVERITY: 🔴 CRITICAL — Supply Chain vulnerability.
```

---

## 📋 S2: GITOPS & DECLARATIVE STATE

### 2.1 Push vs. Pull Deployments (ArgoCD / Flux)
```
WHAT TO CHECK:
  ❑ Does GitHub Actions / GitLab CI have cluster admin credentials to "push" deployments via `kubectl apply`? (Push Model - INSECURE).
  ❑ Is a GitOps operator (ArgoCD, Flux) running INSIDE the cluster, pulling changes from the Git repository? (Pull Model - SECURE).

WHY IT MATTERS:
  If a developer's GitHub account is hacked, and GitHub has cluster admin credentials, the entire production cluster is compromised. In GitOps, the cluster pulls from Git, so the CI/CD platform does not need production credentials.
```

### 2.2 Infrastructure as Code (IaC) Drift & Security
```
WHAT TO CHECK:
  ❑ Are Terraform/CloudFormation templates scanned for misconfigurations BEFORE applying? (e.g., using Checkov, tfsec).
  ❑ Terraform State: Is the `terraform.tfstate` file stored securely (e.g., encrypted S3 bucket with DynamoDB locking) or is it committed to Git? (State files contain ALL cloud secrets in plaintext!).
  ❑ Drift Detection: Is there an automated process checking if someone manually changed a setting in the AWS Console that differs from the Terraform code?

SEVERITY: 🔴 CRITICAL — Exposing `.tfstate` files is equivalent to exposing the root AWS password.
CITATION REQUIRED: Locate the Terraform backend configuration block.
```

---

## 📋 S3: FINOPS & CLOUD COST SECURITY

### 3.1 Financial Denial of Service (FDoS) & Cost Optimization
```
WHAT TO CHECK:
  ❑ Are cloud resources tagged meticulously for billing attribution?
  ❑ Can an attacker trigger a function (e.g., AWS Lambda image processing or massive S3 egress) repeatedly to generate a massive cloud bill?
  ❑ Are billing alarms and hard caps configured?
```

---

## 🛡️ SUPPLY CHAIN & GITOPS MATRIX

| Domain | ❌ Insecure (Fail) | 🟡 Acceptable (Warn) | ✅ SLSA Level 3+ (Pass) |
|--------|-------------------|----------------------|-------------------------|
| **Image Integrity**| Unsigned images | Signed but not verified | Cosign + OPA Gatekeeper |
| **Visibility** | No SBOM | SBOM generated manually | Automated SBOM in Registry |
| **Deployment** | `kubectl` from CI | CI deploys via Helm | GitOps (ArgoCD/Flux Pull) |
| **IaC State** | Local / Git committed | S3 without locking | Encrypted S3 + DynamoDB Lock |

---

## 🚦 PHASE 17 GATE — MANDATORY CHECKLIST

You MUST complete ALL items before proceeding:

```
PHASE 17 GATE CHECKLIST:
  □ [S1.1-1.2] SBOM generation and Image Cryptographic Signing (Cosign) verified
  □ [S2.1] GitOps (Pull model) vs CI (Push model) evaluated
  □ [S2.2] Terraform state security and IaC static analysis verified
  □ [S3.1] Financial DoS (FDoS) protections and billing alarms verified
  □ Minimum 5 citations provided (CI workflows, Terraform backends)
  □ Supply Chain Matrix evaluated
```

### Gate Report Format:

```
═══════════════════════════════════════════════════════
  ✅ PHASE 17 COMPLETE: Supply Chain Security & GitOps
  📊 Findings: [X] Critical | [Y] High | [Z] Medium | [W] Low
  📋 Gate Status: [PASSED/FAILED] ([checked]/5 items)
═══════════════════════════════════════════════════════
```
