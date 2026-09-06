# Phase 19: Pre-Deployment Sentry/Monitoring Validation 🛡️🔍

> **Objective:** This is the ABSOLUTE FINAL gate before deploying to production.
> You will guide the user step-by-step through setting up Sentry for their backend/API,
> performing real-traffic testing, analyzing captured errors, and producing the final deployment verdict.
> Treat the user as NON-TECHNICAL — explain everything in simple terms with exact steps.

---

## ⚠️ WHY THIS PHASE EXISTS

```
Static code analysis (Phases 1-16) catches ~70% of issues.
The remaining ~30% only appear during REAL execution:
  - Runtime errors under actual HTTP requests
  - Database connection failures under load
  - Race conditions that only happen with concurrent users
  - Memory leaks that only appear after hours of running
  - Cloud/infrastructure issues (IAM, networking, DNS)
  - Third-party API failures (payment gateways, email services)

Sentry captures ALL of these in real-time. It is the LAST line of defense.
```

> **IMPORTANT:** Sentry has a FREE tier that is sufficient for this validation.

---

## 🔴 ANTI-LAZINESS RULES FOR THIS PHASE

```
🚫 DO NOT skip this phase — it catches bugs that ALL previous phases missed
🚫 DO NOT fake Sentry results — the user must provide REAL data
🚫 DO NOT analyze without seeing actual Sentry output
🚫 DO NOT rush through setup — each step must be confirmed by the user
✅ WAIT for the user at every step before proceeding
✅ ASK the user to confirm each step is done
✅ EXPLAIN everything in simple non-technical language
✅ PROVIDE exact commands and exact button names to click
```

---

## 📋 SECTION A: SENTRY ACCOUNT & PROJECT SETUP

### Step A1: Create a Sentry Account
```
📝 INSTRUCTIONS TO GIVE THE USER:

1. Open your browser and go to: https://sentry.io/signup/
2. Click "Create Your Account" (the free plan is fine)
3. Sign up with GitHub, Google, or Email
4. After signing up, you'll see the Sentry dashboard

💬 ASK THE USER:
"Have you created your Sentry account and can you see the dashboard?"

⏹️ STOP — Wait for user confirmation.
```

### Step A2: Create a New Project
```
📝 INSTRUCTIONS TO GIVE THE USER:

1. In Sentry dashboard → click "Projects" → "Create Project"
2. Choose your platform:
   - Node.js / Express → select "Node.js" or "Express"
   - Python / Django / Flask / FastAPI → select "Python" then your framework
   - Go → select "Go"
   - Java / Spring Boot → select "Java" then "Spring Boot"
   - PHP / Laravel → select "PHP" then "Laravel"
   - Ruby on Rails → select "Ruby" then "Rails"
   - .NET / ASP.NET → select ".NET" or "ASP.NET Core"
3. Select "Alert me on every new issue"
4. Name your project → Click "Create Project"

💬 ASK THE USER:
"Please share the DSN (long URL) from the setup page."

⏹️ STOP — Wait for DSN.
```

### Step A3: Install Sentry SDK
```
📝 PROVIDE EXACT INSTALLATION COMMANDS BASED ON DETECTED FRAMEWORK:

FOR NODE.JS / EXPRESS:
  npm install @sentry/node
  // At the VERY TOP of your app entry file (before all other imports):
  const Sentry = require("@sentry/node");
  Sentry.init({
    dsn: "YOUR_DSN",
    environment: "pre-deployment-test",
    tracesSampleRate: 1.0,
  });
  // Add error handler AFTER all routes:
  app.use(Sentry.Handlers.errorHandler());

FOR PYTHON (Django/Flask/FastAPI):
  pip install sentry-sdk
  # At the top of your settings/main file:
  import sentry_sdk
  sentry_sdk.init(
      dsn="YOUR_DSN",
      traces_sample_rate=1.0,
      environment="pre-deployment-test",
  )

FOR GO:
  go get github.com/getsentry/sentry-go
  // In main.go:
  err := sentry.Init(sentry.ClientOptions{
      Dsn: "YOUR_DSN",
      Environment: "pre-deployment-test",
      TracesSampleRate: 1.0,
  })

FOR JAVA / SPRING BOOT:
  // Add to pom.xml: io.sentry:sentry-spring-boot-starter:latest
  // In application.properties:
  sentry.dsn=YOUR_DSN
  sentry.environment=pre-deployment-test
  sentry.traces-sample-rate=1.0

FOR PHP / LARAVEL:
  composer require sentry/sentry-laravel
  php artisan sentry:publish --dsn=YOUR_DSN
  // In .env:
  SENTRY_LARAVEL_DSN=YOUR_DSN
  SENTRY_ENVIRONMENT=pre-deployment-test

FOR .NET / ASP.NET CORE:
  dotnet add package Sentry.AspNetCore
  // In Program.cs:
  builder.WebHost.UseSentry(o => {
      o.Dsn = "YOUR_DSN";
      o.Environment = "pre-deployment-test";
      o.TracesSampleRate = 1.0;
  });

💬 ASK: "Have you added Sentry? Does the server start without errors?"
⏹️ STOP — Wait for confirmation.
```

### Step A4: Verify Connection
```
📝 INSTRUCTIONS:
1. Start your server locally or on staging
2. Make ANY API request (even just opening the root URL)
3. Check Sentry dashboard → your project
4. You should see a session or transaction event

💬 ASK: "Can you see any events in Sentry?"
⏹️ STOP — Wait.
```

---

## 📋 SECTION B: REAL-TRAFFIC TESTING SCENARIOS

### Step B1: Normal API Flow
```
📝 INSTRUCTIONS:
Hit ALL your API endpoints as a normal user:
  1. Register a new account (POST /auth/register or equivalent)
  2. Login (POST /auth/login)
  3. Get user profile (GET /users/me)
  4. Create a resource (POST /resource)
  5. Update a resource (PUT /resource/:id)
  6. Delete a resource (DELETE /resource/:id)
  7. List resources with pagination (GET /resources?page=1&limit=10)
  8. Any payment-related endpoints
  9. File upload endpoints (if any)
  10. Logout

You can use Postman, curl, or your frontend app.
⏹️ STOP — Ask if completed.
```

### Step B2: Attack & Edge Case Testing
```
📝 INSTRUCTIONS — Test these EXACT scenarios:

  1. 🔐 AUTH ATTACK: Try accessing protected endpoints WITHOUT a token
  2. 🔐 AUTH ATTACK: Try accessing with an EXPIRED token
  3. 🔐 AUTH ATTACK: Try accessing with a MODIFIED/fake token
  4. 🔑 BOLA ATTACK: Try accessing another user's data (change ID in URL)
  5. 💉 INJECTION: Send this as a name field: ' OR '1'='1'; DROP TABLE users; --
  6. 💉 INJECTION: Send this as a search query: <script>alert(1)</script>
  7. 📦 OVERFLOW: Send a 10MB JSON body to any POST endpoint
  8. 📦 OVERFLOW: Send a request with 1000 query parameters
  9. 🔄 RACE CONDITION: Send the same POST request 20 times simultaneously
  10. ⏱️ TIMEOUT: Disconnect your database → make an API call → reconnect
  11. 💳 PAYMENT: If there's a payment endpoint, send a negative amount
  12. 💳 PAYMENT: If there's a payment endpoint, send amount = 0
  13. 📵 NETWORK: Kill the database connection → check server behavior

⏹️ STOP — Ask if completed.
```

### Step B3: Load Testing (Simple)
```
📝 INSTRUCTIONS:
If you have curl or a tool like "hey" or "ab" (Apache Bench):

  # Install "hey" (simple load tester):
  # macOS: brew install hey
  # Linux: sudo apt install hey
  # Or use: npx autocannon

  # Send 100 requests with 10 concurrent users:
  hey -n 100 -c 10 -H "Authorization: Bearer YOUR_TOKEN" http://localhost:3000/api/resources

This tests:
  - Does the server crash under mild load?
  - Do database connections get exhausted?
  - Do response times degrade?

⏹️ STOP — Ask about results.
```

### Step B4: Wait and Collect
```
📝 INSTRUCTIONS:
Wait 2-3 minutes, then:
1. Open Sentry → your project → "Issues"
2. Tell me what you see
3. For each issue: error title, count, affected endpoint

💬 ASK: "Please share what Sentry shows."
⏹️ STOP — CRITICAL: Wait for real results.
```

---

## 📋 SECTION C: ANALYZING SENTRY RESULTS

### Step C1: Classify Each Finding
```
For EACH Sentry issue:

### [SEVERITY] SENTRY-[N]: [Error Title]

**Error Type:** [UnhandledException / 500 / Timeout / ConnectionError / etc.]
**Occurrences:** [count]
**Affected Endpoint:** [from Sentry breadcrumbs]
**User Impact:** [description]
**Root Cause:** [analysis based on stack trace]
**Fix:** [recommendation]
```

### Step C2: Fix Cycle
```
If Sentry found issues:
1. Add to Phase 17 registry with [SENTRY] prefix
2. Fix via Phase 18
3. Re-test (Section B) → re-check Sentry
4. Repeat until ZERO Critical/High from Sentry
```

---

## 📋 SECTION D: FINAL DEPLOYMENT VERDICT

```markdown
## 🏁 Phase 19: Pre-Deployment Sentry Validation — FINAL REPORT

| Metric | Value |
|--------|-------|
| Sentry Connected | ✅ YES |
| Normal API Flow | ✅ PASSED / ❌ FAILED |
| Attack Testing | ✅ PASSED / ❌ FAILED |
| Load Testing | ✅ PASSED / ❌ FAILED |
| Total Sentry Issues | [count] |
| Critical Remaining | [0 or count] |
| High Remaining | [0 or count] |

> **SENTRY VERDICT: [🟢 CLEAN / 🟡 ACCEPTABLE / 🔴 NOT READY]**

┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   ✅ BACKEND IS READY FOR PRODUCTION DEPLOYMENT             │
│                                                             │
│   ✓ Statically analyzed (16 phases, [N] citations)         │
│   ✓ Fresh-Eyes re-analyzed (Rule 9)                        │
│   ✓ Remediated with verified fixes (Phase 18)              │
│   ✓ Real-traffic tested with Sentry (Phase 19)             │
│   ✓ All Critical/High issues resolved                      │
│                                                             │
│   Signed off at: [Date/Time]                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘

Keep Sentry active in production.
Set up alerts for:
  - Any new 500 error
  - Error rate > 1% of requests
  - Response time > 2 seconds
Monitor the dashboard daily for the first week after deployment.
```

---

## 🚦 PHASE 19 GATE — MANDATORY CHECKLIST

```
PHASE 19 GATE CHECKLIST:
  □ [A1-A4] Sentry setup and verified
  □ [B1] Normal API flow tested
  □ [B2] Attack/edge case testing completed
  □ [B3] Load testing done
  □ [B4] Sentry data collected
  □ [C1] Issues classified
  □ [C2] Critical/High fixed (if any)
  □ [D] Final verdict produced
```

### ⛔ FINAL STOP POINT
**This is the END of the entire audit process.**
- **✅ READY** → Deploy to production with confidence
- **❌ NOT READY** → Back to Phase 18, fix, re-test
