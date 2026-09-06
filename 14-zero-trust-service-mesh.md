# Phase 16: Zero Trust Architecture & Service Mesh 🕸️

> **Objective:** Evaluate internal network security. The legacy assumption that "inside the VPC is safe" is dead. This phase audits the implementation of Zero Trust Architecture (ZTA) and Service Mesh technologies to ensure that every microservice verifies the identity and authorization of every other microservice.

---

## 📋 Z1: SERVICE MESH & mTLS CHECKS

### 1.1 Mutual TLS (mTLS) Enforcement
```
WHAT TO CHECK:
  ❑ Is a Service Mesh (Istio, Linkerd, Consul Connect) installed in the Kubernetes cluster?
  ❑ Is mTLS enforced strictly (`STRICT` mode in Istio) or is it in permissive mode?
  ❑ Does traffic between `pod-a` and `pod-b` travel in plaintext?

WHY IT MATTERS:
  If an attacker breaches a low-priority pod (e.g., `PDF-generator`) and installs a packet sniffer, they can capture plaintext HTTP traffic containing session tokens and passwords flowing to the database. mTLS encrypts all pod-to-pod traffic.

SEVERITY: 🔴 CRITICAL — Internal network traffic interception.
CITATION REQUIRED: Find the `PeerAuthentication` (Istio) or Linkerd annotations enforcing mTLS.
```

### 1.2 Workload Identity (SPIFFE/SPIRE)
```
WHAT TO CHECK:
  ❑ How do services prove who they are? Do they use static API keys (BAD) or cryptographically verifiable identities (GOOD)?
  ❑ Is the SPIFFE (Secure Production Identity Framework for Everyone) standard implemented?
  ❑ Are short-lived X.509 certificates automatically rotated for every pod?
```

---

## 📋 Z2: MICRO-SEGMENTATION & AUTHORIZATION

### 2.1 Service-to-Service Authorization
```
WHAT TO CHECK:
  ❑ Does the architecture rely solely on Kubernetes NetworkPolicies (Layer 4 IP/Port blocking)?
  ❑ Are Layer 7 AuthorizationPolicies implemented? (e.g., Service A can GET `/users`, but cannot POST `/users`).
  ❑ Can the `reporting-service` access the `/delete` endpoint of the `billing-service`?

ANTI-PATTERN EXAMPLE:
  🔴 The internal network is flat. Once inside, any service can call any endpoint on any other service. 
  ✅ Fix: Enforce Istio `AuthorizationPolicy` to restrict paths and methods between specific service accounts.
```

### 2.2 Egress Traffic Control (Outbound)
```
WHAT TO CHECK:
  ❑ Are pods allowed to make outbound internet connections freely?
  ❑ Is a Service Mesh Egress Gateway configured to block all outbound traffic except to explicit allowlisted domains (e.g., `api.stripe.com`)?

WHY IT MATTERS:
  If a pod is compromised and an attacker tries to establish a Reverse Shell or download malware from `hacker.com`, a strict Egress Gateway will block the connection, neutralizing the attack.

SEVERITY: 🟠 HIGH — Lack of egress control guarantees successful data exfiltration.
```

---

## 📋 Z3: SECRETS ENGINE & DYNAMIC CREDENTIALS

### 3.1 Dynamic Secrets (HashiCorp Vault)
```
WHAT TO CHECK:
  ❑ Are database credentials static (e.g., valid for 5 years)?
  ❑ Is a Secrets Engine (like Vault) generating *Dynamic Secrets*? (e.g., When the backend connects to Postgres, Vault generates a username/password that self-destructs after 15 minutes).

SEVERITY: 🟠 HIGH — Static secrets leaked in logs or Slack are the primary cause of massive breaches.
```

---

## 🛡️ ZERO TRUST COMPLIANCE MATRIX

| Domain | ❌ Legacy (Insecure) | 🟡 Transitional | ✅ Zero Trust (Secure) |
|--------|----------------------|-----------------|------------------------|
| **Pod Traffic** | Plaintext HTTP | Permissive mTLS | Strict mTLS (Istio/Linkerd) |
| **Service Auth** | Shared API Keys | K8s NetworkPolicies | Layer 7 AuthorizationPolicies |
| **Identity** | IP Addresses | K8s Service Accounts | SPIFFE / SPIRE X.509 certs |
| **Egress** | Fully Open (0.0.0.0/0) | Blocked by IP | Layer 7 Egress Gateway Allowlist |

---

## 🚦 PHASE 16 GATE — MANDATORY CHECKLIST

You MUST complete ALL items before proceeding:

```
PHASE 14 GATE CHECKLIST:
  □ [Z1.1-1.2] Service Mesh mTLS and Workload Identity (SPIFFE) verified
  □ [Z2.1-2.2] Layer 7 Authorization and Egress Gateway controls verified
  □ [Z3.1] Dynamic Secrets Engine (Vault) usage checked
  □ Minimum 5 infrastructure configuration citations provided
  □ Zero Trust Compliance Matrix evaluated
```

### Gate Report Format:

```
═══════════════════════════════════════════════════════
  ✅ PHASE 14 COMPLETE: Zero Trust Architecture & Mesh
  📊 Findings: [X] Critical | [Y] High | [Z] Medium | [W] Low
  📋 Gate Status: [PASSED/FAILED] ([checked]/5 items)
═══════════════════════════════════════════════════════
```

> ⏹️ **STOP POINT** — Output Phase 14 report. Wait for user before Phase 15.
