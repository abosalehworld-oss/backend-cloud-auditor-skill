# Phase 2: Business Logic & Domain Integrity 🧠

> **Objective:** Backend systems handle money, permissions, and complex state changes. A failure in business logic is rarely caught by automated security scanners, but it destroys businesses.

---

## 📋 BUSINESS LOGIC CHECKS

### CHECK BIZ-1: Currency & Float Math Inaccuracies
```
WHAT TO CHECK:
  ❑ Is money, weight, or precise billing calculated using standard `float` or `double` data types?
  ❑ Are floating-point rounding errors possible?
  
WHY IT MATTERS:
  `0.1 + 0.2 = 0.30000000000000004`. Using floats for financial transactions leads to gradual loss of money or failed reconciliation.
  
REQUIRED FIX:
  Must use specialized Decimal/BigDecimal types, or store money as Integers (cents).

SEARCH PATTERNS:
  - `price: Number`, `amount: float`
  - `total = price * tax_rate` (using standard math operators instead of Decimal libraries)
  
SEVERITY: 🔴 CRITICAL — Financial loss.
```

### CHECK BIZ-2: State Machine Integrity
```
WHAT TO CHECK:
  ❑ Do entities (Orders, Users, Payments) transition through explicit, validated states?
  ❑ Can an API request force an invalid state transition? (e.g., Transitioning an order from `PENDING` directly to `REFUNDED` bypassing `PAID` and `SHIPPED`).

COMMON BUGS:
  🔴 The controller accepts `{"status": "DELIVERED"}` from the client and updates the DB without verifying if the current state was `SHIPPED`.

CITATION REQUIRED: Find the Order/Payment update logic and verify state transition constraints.
```

### CHECK BIZ-3: Edge Cases & Boundary Conditions
```
WHAT TO CHECK:
  ❑ Negative Values: Can a user transfer `-500` dollars, effectively stealing from the recipient?
  ❑ Zero Values: What happens if a division by zero occurs in billing calculations?
  ❑ Maximum Limits: Is there a maximum cap on item quantities? (e.g., ordering 2 billion items, causing integer overflow in the total price).

SEARCH PATTERNS:
  - Missing `if (amount <= 0) throw Error`
  - `update balance = balance + ?`
  
SEVERITY: 🔴 CRITICAL
```

### CHECK BIZ-4: Date, Time & Timezones
```
WHAT TO CHECK:
  ❑ Are dates stored in UTC in the database?
  ❑ Is timezone conversion handled strictly at the edge (presentation layer)?
  ❑ Are scheduling jobs (Cron) timezone-aware?

COMMON BUGS:
  🔴 `new Date()` is stored directly in the DB. When the server moves from a US region to an EU region, all timestamps shift by 6 hours.
```

---

## 🚦 PHASE 2 GATE — MANDATORY CHECKLIST

```
PHASE 2 GATE CHECKLIST:
  □ [BIZ-1] Currency and floating-point math verified
  □ [BIZ-2] State machine transition validations checked
  □ [BIZ-3] Negative/Zero boundary values checked
  □ [BIZ-4] UTC Date storage verified
  □ Minimum 6 code citations provided
```

### Gate Report Format:
```
═══════════════════════════════════════════════════════
  ✅ PHASE 2 COMPLETE: Business Logic & Integrity
  📊 Findings: [X] Critical | [Y] High | [Z] Medium | [W] Low
  📋 Gate Status: [PASSED/FAILED] ([checked]/5 items)
═══════════════════════════════════════════════════════
```
### ⛔ STOP POINT: Wait for user acknowledgment.
