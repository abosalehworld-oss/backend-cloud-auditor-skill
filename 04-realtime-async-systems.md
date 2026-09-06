# Phase 4: Real-time & Asynchronous Systems ⚡

> **Objective:** Real-time protocols (WebSockets, SSE) and async Webhooks bypass traditional REST API security and performance guards. This phase audits the resilience of real-time communication.

---

## 📋 REAL-TIME PROTOCOL CHECKS

### CHECK ASYNC-1: WebSocket Authentication & AuthZ
```
WHAT TO CHECK:
  ❑ How are WebSocket connections authenticated? (Tokens in URL query params are BAD, headers are GOOD but hard in browsers).
  ❑ Does the server re-validate authorization upon subscribing to specific channels? (Just because you are connected to the socket doesn't mean you can listen to "Admin_Channel").
  
SEARCH PATTERNS:
  - `io.on('connection')` (Socket.io)
  - `ws.on('message')`

SEVERITY: 🔴 CRITICAL — WebSocket hijacking exposes continuous data streams.
```

### CHECK ASYNC-2: WebSocket Resource Exhaustion (Memory Leaks)
```
WHAT TO CHECK:
  ❑ Are ping/pong heartbeats implemented to detect dead connections (e.g., mobile phone lost battery)?
  ❑ Are WebSocket objects properly deleted from memory maps/arrays when the connection drops?

COMMON BUGS:
  🔴 A user connects and disconnects 50 times due to bad signal. The server keeps 50 "zombie" connections in memory. Memory slowly hits 100% and crashes.
```

### CHECK ASYNC-3: Webhook Security & Signature Verification
```
WHAT TO CHECK:
  ❑ When the backend receives Webhooks from Third-Parties (Stripe, GitHub, Twilio):
  ❑ Does the endpoint verify the HMAC signature provided in the headers?
  ❑ Is the endpoint susceptible to Replay Attacks? (Does it check the timestamp included in the webhook payload?)

WEBHOOK SECURITY MATRIX:
  ┌─────────────────────────┬──────────────────────────┬──────────────────────┐
  │ Verification Method     │ ❌ INSECURE                │ ✅ SECURE              │
  ├─────────────────────────┼──────────────────────────┼──────────────────────┤
  │ Origin Verification     │ Checks IP Address        │ Validates HMAC Sign  │
  │ Replay Protection       │ None                     │ Checks Timestamp     │
  │ Payload Handling        │ JSON parsed before auth  │ Raw body used for sig│
  └─────────────────────────┴──────────────────────────┴──────────────────────┘

SEVERITY: 🔴 CRITICAL — An attacker can POST to `/webhooks/stripe` with a fake payload saying "User X paid $1000", bypassing payment entirely.
CITATION REQUIRED: Find all webhook receiving endpoints and verify HMAC.
```

### CHECK ASYNC-4: Server-Sent Events (SSE) & Connection Holding
```
WHAT TO CHECK (If SSE is used):
  ❑ How many simultaneous open connections can the server handle? (Node.js/Go are great, but traditional threading models like Apache/PHP will exhaust worker threads instantly).
  ❑ Is there a reverse proxy (Nginx) killing connections due to timeout?
```

---

## 🚦 PHASE 4 GATE — MANDATORY CHECKLIST

```
PHASE 4 GATE CHECKLIST:
  □ [ASYNC-1] WebSocket Auth and Channel AuthZ verified
  □ [ASYNC-2] WebSocket memory leak / heartbeat checks verified
  □ [ASYNC-3] Webhook HMAC signature and replay protection verified
  □ [ASYNC-4] SSE thread exhaustion checked
  □ Minimum 5 code citations provided
```

### Gate Report Format:
```
═══════════════════════════════════════════════════════
  ✅ PHASE 4 COMPLETE: Real-time & Async Systems
  📊 Findings: [X] Critical | [Y] High | [Z] Medium | [W] Low
  📋 Gate Status: [PASSED/FAILED] ([checked]/5 items)
═══════════════════════════════════════════════════════
```
### ⛔ STOP POINT: Wait for user acknowledgment.
