# Phase 11: Observability & Auditability 🔭

> **Objective:** If the server is hacked, or if it crashes under load, do we have the telemetry to know *why*? This phase ensures logging, metrics, and tracing are comprehensive, secure, and devoid of sensitive user data.

---

## 📋 OBSERVABILITY CHECKS

### CHECK OBS-1: Distributed Tracing & Correlation IDs
```
WHAT TO CHECK (Microservices):
  ❑ Is a `Trace-ID` or `Correlation-ID` generated at the API Gateway (or read from the incoming request)?
  ❑ Is this ID passed in the headers to EVERY downstream microservice?
  ❑ Is this ID included in every single log statement?

WHY IT MATTERS:
  In a microservices architecture, a user clicks "Checkout". Service A calls Service B, which calls Service C. Service C fails. Without a Trace ID, it is impossible to stitch together the logs across 3 different servers to figure out what happened.

SEVERITY: 🟠 HIGH — Makes debugging distributed systems nearly impossible.
CITATION REQUIRED: Find where HTTP headers are parsed and trace IDs are injected into the logger context.
```

### CHECK OBS-2: Structured Logging
```
WHAT TO CHECK:
  ❑ Are logs output as plain text strings or structured JSON?
  ❑ Are Log Levels used appropriately? (DEBUG, INFO, WARN, ERROR, FATAL).
  ❑ Are unhandled exceptions caught globally and logged with full stack traces?

SEARCH PATTERNS:
  - `console.log('User logged in: ' + user.id)` (BAD)
  - `logger.info({ message: 'User logged in', userId: user.id })` (GOOD)
```

### CHECK OBS-3: PII & Secrets Leakage in Logs
```
WHAT TO CHECK:
  ❑ Does the HTTP request logger blindly log the entire `req.body`?
  ❑ Are explicit filters/masking functions used to scrub passwords, credit card numbers, SSNs, and JWTs before they hit the logging stdout?

COMMON BUGS:
  🔴 A user attempts to login but fails. The logger dumps: `Failed login attempt: {"email":"test@x.com", "password":"MySecretPassword123"}`. The plaintext password is now permanently stored in Splunk/Datadog.

SEVERITY: 🔴 CRITICAL — Instant GDPR/CCPA violation.
CITATION REQUIRED: Review the HTTP logging middleware (Morgan, Winston, Serilog, Logback).
```

### CHECK OBS-4: Health Probes & Metrics (Prometheus/Grafana)
```
WHAT TO CHECK:
  ❑ Liveness Probe (`/healthz`): Returns 200 OK if the application is running. (If it fails, Kubernetes kills the pod).
  ❑ Readiness Probe (`/readyz`): Checks if the DB and Redis are connected. (If it fails, K8s stops sending traffic, but doesn't kill the pod).
  ❑ Are RED metrics (Rate, Errors, Duration) exposed for Prometheus scraping?

COMMON BUGS:
  🔴 The Liveness probe checks the Database. When the DB gets slow, the Liveness probe times out. Kubernetes assumes the app is dead and kills the pod. Now ALL pods are killed, causing a total outage, just because the DB was slow. (Liveness should NEVER check external dependencies).
```

---

## 🚦 PHASE 11 GATE — MANDATORY CHECKLIST

```
PHASE 11 GATE CHECKLIST:
  □ [OBS-1] Correlation/Trace IDs verified across service boundaries
  □ [OBS-2] Structured JSON logging verified
  □ [OBS-3] PII and Secret masking verified in HTTP loggers
  □ [OBS-4] Liveness/Readiness probes correctly separated
  □ Minimum 5 code citations provided
```

### Gate Report Format:
```
═══════════════════════════════════════════════════════
  ✅ PHASE 11 COMPLETE: Observability & Auditability
  📊 Findings: [X] Critical | [Y] High | [Z] Medium | [W] Low
  📋 Gate Status: [PASSED/FAILED] ([checked]/5 items)
═══════════════════════════════════════════════════════
```
### ⛔ STOP POINT: Wait for user acknowledgment.
