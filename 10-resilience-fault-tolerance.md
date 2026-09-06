# Phase 10: Resilience & Fault Tolerance 🛡️

> **Objective:** Systems will fail. Databases will go down. Third-party APIs will timeout. This phase evaluates if the backend fails *gracefully* or if a single external failure causes a catastrophic cascading outage across the entire architecture.

---

## 📋 FAULT TOLERANCE CHECKS

### CHECK FAULT-1: Circuit Breakers for External Calls
```
WHAT TO CHECK:
  ❑ Are HTTP calls to external APIs (e.g., Stripe, SendGrid, Microservice B) protected by a Circuit Breaker?
  ❑ If the external API hangs (doesn't respond for 30 seconds), does the backend thread hang with it?

WHY IT MATTERS:
  If a payment API goes down and hangs, 1000 users trying to checkout will freeze 1000 backend worker threads. Within 1 minute, the entire backend is unresponsive to *all* users (Cascading Failure).
  
REQUIRED FIX:
  Implement Circuit Breakers (e.g., Opossum in Node.js, Hystrix/Resilience4j in Java). After X timeouts, the circuit "opens" and immediately fails fast, preventing thread starvation.

SEVERITY: 🔴 CRITICAL — Immediate DoS during third-party outages.
CITATION REQUIRED: Find external API calls and verify timeout/circuit breaker logic.
```

### CHECK FAULT-2: Retry Mechanisms & Exponential Backoff
```
WHAT TO CHECK:
  ❑ Are failed network requests retried automatically?
  ❑ Is Exponential Backoff used? (e.g., wait 1s, then 2s, then 4s, then 8s).
  ❑ Is Jitter (randomness) added to the backoff?

COMMON BUGS:
  🔴 10,000 mobile apps lose connection to the server. The server reboots. All 10,000 apps immediately retry at the EXACT same millisecond. The server crashes again (Thundering Herd attack). Jitter prevents this.
```

### CHECK FAULT-3: Bulkheads & Resource Isolation
```
WHAT TO CHECK:
  ❑ Are connection pools or thread pools segregated?
  ❑ If the "Report Generation" feature exhausts database connections, does it kill the "User Login" feature?

REQUIRED ARCHITECTURE:
  ✅ Separate connection pools for heavy read/analytical queries vs fast transactional queries. If reports fail, logins still work.
```

### CHECK FAULT-4: Graceful Shutdown (SIGTERM Handling)
```
WHAT TO CHECK:
  ❑ When a deployment happens (Docker/K8s sends SIGTERM), does the server aggressively kill active connections?
  ❑ Does it stop accepting new requests, finish processing active requests, and then exit gracefully?

SEARCH PATTERNS:
  - `process.on('SIGTERM')`
  - `server.close()`
  
SEVERITY: 🟠 HIGH — Causes dropped payments during every CI/CD deployment.
```

---

## 🚦 PHASE 10 GATE — MANDATORY CHECKLIST

```
PHASE 10 GATE CHECKLIST:
  □ [FAULT-1] Circuit Breakers verified on 3rd party calls
  □ [FAULT-2] Retries with Exponential Backoff and Jitter verified
  □ [FAULT-3] Bulkheads and isolated connection pools checked
  □ [FAULT-4] Graceful shutdown (SIGTERM) handling verified
  □ Minimum 5 code citations provided
```

### Gate Report Format:
```
═══════════════════════════════════════════════════════
  ✅ PHASE 10 COMPLETE: Resilience & Fault Tolerance
  📊 Findings: [X] Critical | [Y] High | [Z] Medium | [W] Low
  📋 Gate Status: [PASSED/FAILED] ([checked]/5 items)
═══════════════════════════════════════════════════════
```
### ⛔ STOP POINT: Wait for user acknowledgment.
