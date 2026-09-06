# Phase 1: Architecture & Topology Review 🏗️

> **Objective:** Analyze the backend's foundational structure, domain boundaries, and component coupling. A flawed topology (e.g., a distributed monolith) creates cascading failures that no amount of code-level optimization can fix.

---

## 🔍 PRE-CHECK: Topology Map

### 1.1 Project Structure & Bounded Contexts
```
ACTION: Analyze the directory structure to determine the architectural pattern.
LOOK FOR:
  - Is it a Monolith, Modular Monolith, or Microservices architecture?
  - Are Domain boundaries respected? (e.g., `users/`, `orders/`, `payments/` directories).
  - Hexagonal/Clean Architecture: Are Ports and Adapters explicitly separated from Business Logic?
  - MVC: Are controllers bloated with business logic?
```

### 1.2 Dependency Injection (DI) & Inversion of Control
```
ACTION: Review how services are instantiated and passed.
CHECK:
  ❑ Are dependencies hardcoded inside classes (tight coupling)?
  ❑ Is a DI container used? (e.g., TSyringe, Inversify, Spring Context, Google Wire).
  ❑ Are singletons abused to hold mutable state?
```

---

## 📋 ARCHITECTURE CHECKS

### CHECK ARCH-1: Controller Bloat & Separation of Concerns
```
WHAT TO CHECK:
  ❑ Do API Controllers/Routers contain direct database calls (`SQL` or `ORM` queries)?
  ❑ Do Controllers contain complex formatting or calculations?
  
WHY IT MATTERS:
  Controllers should ONLY extract HTTP parameters, call a Service/Use-Case layer, and format the HTTP response. Business logic inside controllers makes the system un-testable and impossible to expose via other protocols (like gRPC or WebSockets).

SEVERITY: 🟠 HIGH — Major tech debt.
CITATION REQUIRED: Find the largest controller file and measure its lines of code.
```

### CHECK ARCH-2: The Distributed Monolith (Microservices Anti-Pattern)
```
WHAT TO CHECK (If Microservices are used):
  ❑ Do microservices share the EXACT same database? (Integration Database Anti-Pattern).
  ❑ Do microservices make deep, synchronous HTTP calls to each other? (Service A calls B, which calls C... if C fails, everything fails).
  ❑ Is there a lack of an API Gateway? (Clients calling microservices directly).

MICROSERVICES HEALTH MATRIX:
  ┌─────────────────────────┬──────────────────────────┬──────────────────────┐
  │ Architectural Trait     │ ❌ DISTRIBUTED MONOLITH    │ ✅ TRUE MICROSERVICES  │
  ├─────────────────────────┼──────────────────────────┼──────────────────────┤
  │ Data Ownership          │ Shared Database Tables   │ Database per Service │
  │ Communication           │ Synchronous REST chains  │ Async Events/PubSub  │
  │ Client Interaction      │ Direct IPs / DNS         │ API Gateway          │
  │ Deployment              │ Lockstep (Deployed together)│ Independent Release │
  └─────────────────────────┴──────────────────────────┴──────────────────────┘

SEVERITY: 🔴 CRITICAL — Defeats the purpose of distributed systems.
```

### CHECK ARCH-3: Configuration & Environment Management
```
WHAT TO CHECK:
  ❑ Is configuration hardcoded in the codebase?
  ❑ Are 12-Factor App principles followed? (Config stored in environment).
  ❑ Is there a centralized config validation mechanism? (e.g., crashing on startup if `DATABASE_URL` is missing, rather than crashing later during a user request).

SEARCH PATTERNS:
  - `process.env.XXX` scattered across 100 files instead of a single `config.ts` validator (Node).
  - `os.environ.get("XXX")` without fallbacks or type checking (Python).
```

---

## 🚦 PHASE 1 GATE — MANDATORY CHECKLIST

```
PHASE 1 GATE CHECKLIST:
  □ [ARCH-1] Controller vs Service separation verified
  □ [ARCH-2] Microservices coupling and DB ownership checked
  □ [ARCH-3] Centralized configuration validation checked
  □ Minimum 5 code citations provided
  □ Files examined list produced
```

### Gate Report Format:
```
═══════════════════════════════════════════════════════
  ✅ PHASE 1 COMPLETE: Architecture & Topology
  📊 Findings: [X] Critical | [Y] High | [Z] Medium | [W] Low
  📋 Gate Status: [PASSED/FAILED] ([checked]/5 items)
═══════════════════════════════════════════════════════
```
### ⛔ STOP POINT: Wait for user acknowledgment.
