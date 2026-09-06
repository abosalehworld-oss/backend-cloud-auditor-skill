# Phase 5: Database & Data Architecture 🗄️

> **Objective:** Evaluate the data layer for consistency, scalability, and performance. In distributed systems, data integrity (ACID vs Eventual Consistency) and transaction management are the most complex engineering challenges.

---

## 📋 DATA ARCHITECTURE CHECKS

### CHECK DATA-1: Distributed Transactions (Saga Pattern vs 2PC)
```
WHAT TO CHECK:
  If the architecture uses Microservices:
  ❑ How are transactions handled across multiple databases?
  ❑ Is there a Two-Phase Commit (2PC)? (Anti-pattern: Locks resources too long).
  ❑ Is the Saga Pattern (Choreography or Orchestration) implemented for long-running transactions?
  ❑ Are compensating transactions (Rollbacks) explicitly defined if a step fails?

COMMON BUGS:
  🔴 Microservice A deducts user balance. Microservice B fails to ship the item. Because they use separate databases, there is no automatic SQL rollback. The user loses money.

SEVERITY: 🔴 CRITICAL — Leads to permanent data corruption and financial loss.
CITATION REQUIRED: Find a multi-service transaction and trace its failure path.
```

### CHECK DATA-2: Database Connection Management
```
WHAT TO CHECK:
  ❑ Is a Connection Pool configured? (e.g., PgBouncer, HikariCP, Prisma pool).
  ❑ Are connections properly released back to the pool after use?
  ❑ In Serverless environments (AWS Lambda), is connection pooling handled outside the function? (Otherwise, every request opens a new connection, crashing the DB).

SEARCH PATTERNS:
  - `max_connections` config limits.
  - `pool: { min: 2, max: 10 }`
```

### CHECK DATA-3: Query Optimization & N+1 Problems
```
WHAT TO CHECK:
  ❑ Does the ORM fetch relationships in a loop instead of using eager loading (`JOIN`, `populate`, `select_related`)?
  ❑ Are foreign keys correctly indexed? (Missing indexes cause full-table scans).

SEARCH PATTERNS:
  - Nested `for` loops making DB queries.
  - `User.posts.map(post => post.comments)`

SEVERITY: 🟠 HIGH — Kills database performance as tables grow.
```

### CHECK DATA-4: Sharding & Multi-Tenancy Architecture
```
WHAT TO CHECK:
  If the application is B2B SaaS (Multi-Tenant):
  ❑ How is tenant isolation achieved? (Database-per-tenant, Schema-per-tenant, or Row-level security).
  ❑ Can a query accidentally leak Tenant A's data to Tenant B because a `WHERE tenant_id = ?` was forgotten?

SEVERITY: 🔴 CRITICAL — Data leakage between corporate clients is a catastrophic breach.
```

---

## 🚦 PHASE 5 GATE — MANDATORY CHECKLIST

```
PHASE 5 GATE CHECKLIST:
  □ [DATA-1] Distributed Transactions (Saga) verified
  □ [DATA-2] Connection pooling configured correctly
  □ [DATA-3] N+1 Queries and missing indexes searched
  □ [DATA-4] Multi-tenancy isolation verified
  □ Minimum 5 code citations provided
```

### Gate Report Format:
```
═══════════════════════════════════════════════════════
  ✅ PHASE 5 COMPLETE: Database & Data Architecture
  📊 Findings: [X] Critical | [Y] High | [Z] Medium | [W] Low
  📋 Gate Status: [PASSED/FAILED] ([checked]/5 items)
═══════════════════════════════════════════════════════
```
### ⛔ STOP POINT: Wait for user acknowledgment.
