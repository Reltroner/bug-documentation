# 🔐 Reltroner ERP

## Phase 3 — Gateway → ERP Module Access

### 📍 CHECKPOINT #1 (IN PROGRESS)

---

## 0. Executive Snapshot

| Item                      | Status                         |
| ------------------------- | ------------------------------ |
| Phase 2 (SSO Keycloak)    | ✅ FIXED & FROZEN               |
| Phase 3 Core Architecture | 🟡 IMPLEMENTED (Checkpoint #1) |
| Gateway → Finance Flow    | 🟡 WORKING (Local)             |
| Security Boundary         | ✅ CLEAN                        |
| Deployment                | ❌ NOT YET                      |
| Phase 3 Completion        | ❌ NOT COMPLETE                 |

---

## 1. Phase 3 Objective (Non-Negotiable)

Phase 3 exists to **connect the Auth Gateway (`app.reltroner.com`) to ERP modules (starting with `finance.reltroner.com`)** using a **temporary, scoped token (RMAT)**.

### Core Goals

* Gateway issues access authority
* ERP modules consume access passively
* No duplication of authentication
* No violation of Phase 2 guarantees

### Non-Negotiable Principles

* **Gateway = authentication authority & token issuer**
* **Module = passive consumer**
* ❌ No shared session across domains
* ❌ No RBAC
* ❌ No user provisioning
* ❌ Phase 2 must remain untouched

---

## 2. Global Architecture (Current)

```
Browser
  │
  ▼
app.reltroner.com
(Auth Gateway)
  │  - Phase 2 SSO session
  │  - RMAT generation
  │
  ▼ redirect
finance.reltroner.com/sso/consume
  │  - RMAT verification
  │  - Finance-local session
  ▼
Finance Dashboard
```

---

## 3. Repositories & Responsibilities (Checkpoint #1)

### 3.1 `app.reltroner.com` — Auth Gateway

**Responsibilities**

* Complete SSO via Keycloak (Phase 2)
* Maintain `sso_authenticated` session
* Generate RMAT (Phase 3)
* Redirect users to ERP modules

**Explicitly Does NOT**

* Log users into database
* Run ERP logic
* Implement RBAC
* Provision users

---

### 3.2 `finance.reltroner.com` — Finance ERP Module

**Responsibilities**

* Provide `/sso/consume` endpoint
* Verify RMAT integrity & claims
* Create finance-local session
* Execute Finance business logic

**Explicitly Does NOT**

* Login to Keycloak
* Generate tokens
* Route to other modules

---

## 4. Phase 2 Status (SSO — FINAL & FROZEN)

### 4.1 SSO Flow (Locked)

```
User → /login
     → /sso/login
     → Keycloak
     → /sso/callback
```

Gateway session created:

```php
session([
  'sso_authenticated' => true,
  'access_token' => '...',
  'id_token' => '...',
]);
```

---

### 4.2 Critical Bug Resolved

**Bug:** `403 Invalid SSO state`

**Root Cause**

* Session not regenerated before redirect to Keycloak
* `sso_state` lost during callback

**Fix**

```php
$request->session()->regenerate();
```

**Status:** ✅ FIXED
**Note:** This was a Phase 2 lifecycle issue, not a Phase 3 bug.

---

## 5. Phase 3 Core Implementation (Checkpoint #1)

### 5.1 RMAT — Reltroner Module Access Token

**Characteristics**

* JWT
* HS256
* TTL ≤ 60 seconds
* Single-module scope
* Single-use intention
* Not persisted
* Not logged

**Final Payload Schema**

```json
{
  "iss": "https://app.reltroner.com",
  "aud": "finance.reltroner.com",
  "sub": "<keycloak_sub>",
  "email": "<email|null>",
  "iat": 1737000000,
  "exp": 1737000060,
  "jti": "<uuid>",
  "ctx": {
    "module": "finance",
    "entry": "dashboard"
  }
}
```

---

### 5.2 Gateway — Module Router

**Endpoint**

```
GET /modules/finance
```

**Controller**

```
FinanceRedirectController
```

**Flow**

* Ensure `sso_authenticated`
* Generate RMAT
* Redirect to:

```
finance.reltroner.com/sso/consume?token=...
```

---

### 5.3 Gateway — `ModuleTokenFactory`

**Single Responsibility**

* Issue RMAT
* Define:

  * issuer
  * audience
  * TTL
  * context

**Explicitly Does NOT**

* Verify tokens
* Log tokens
* Write to database

---

### 5.4 Gateway Configuration (Phase 3)

```php
'gateway' => [
  'issuer',
  'signing_key'
],

'modules' => [
  'finance' => 'https://finance.reltroner.com'
]
```

---

## 6. Finance Module — Phase 3 Implementation

### 6.1 `/sso/consume` Endpoint

**Controller**

```
ConsumeController
```

**Flow**

1. Extract token from query
2. Verify signature
3. Validate claims:

   * issuer
   * audience
   * expiration
   * TTL
   * `ctx.module`
4. Create finance-local session:

```php
session([
  'finance_authenticated' => true,
  'external_id' => sub
]);
```

5. Redirect to `/dashboard`

---

### 6.2 Finance Middleware

**Middleware**

```
EnsureGatewayAuthenticated
```

**Function**

* Block access without finance session
* Redirect to Gateway login

---

### 6.3 Finance Configuration (Phase 3)

```php
'gateway' => [
  'issuer',
  'audience',
  'signing_key',
  'login_url'
]
```

Finance has **no knowledge of other modules**.

---

## 7. Boundary Enforcement (Cleaned)

### ❌ Removed from Finance

* `FinanceRedirectController`
* `ModuleTokenFactory`
* `services.modules.finance`

### ✅ Exists ONLY in Gateway

* Module routing
* Token issuance

**Phase 3 boundary is clean and enforced.**

---

## 8. Testing Status (Checkpoint #1)

### 8.1 Passing (Local)

* Gateway login via Keycloak
* RMAT generation
* Redirect to Finance
* Finance session creation
* Finance dashboard accessible

---

### 8.2 Not Yet Tested (Required for Checkpoint #2)

* Expired token (>60s)
* Modified token
* Token replay
* Multi-tab race conditions
* Logout propagation

---

## 9. Deployment Status

| Item        | Status   |
| ----------- | -------- |
| Docker      | 🟡 Draft |
| VPS         | ❌        |
| Hostinger   | ❌        |
| HTTPS       | ❌        |
| Real domain | ❌        |

➡️ **Phase 3 must not be considered complete.**

---

## 10. Intentionally Deferred Work

| Item              | Reason  |
| ----------------- | ------- |
| RBAC              | Phase 4 |
| User provisioning | Phase 4 |
| Refresh tokens    | Phase 4 |
| Global logout     | Phase 4 |
| Replay protection | Phase 4 |

These are **conscious architectural decisions**, not technical debt.

---

## 11. Known Risks (Checkpoint #1)

| Risk         | Status            |
| ------------ | ----------------- |
| Token replay | 🟡 Accepted       |
| No RBAC      | 🟡 Accepted       |
| Local-only   | 🔴 Not prod-ready |

---

## 12. CHECKPOINT #1 — Conclusion

Phase 3 has reached its **first stable architectural checkpoint**:

* Core flow works
* Boundaries are correct
* Security model is valid

However:

* Deployment
* Hardening
* Negative testing

are **not yet complete**.

This checkpoint **must exist** before moving forward.

---

## 13. What Defines CHECKPOINT #2 (Not Started)

Checkpoint #2 is valid only if:

* End-to-end negative tests pass
* Docker Compose is stable
* HTTPS is enabled
* Real domains are used
* Gateway & Finance run on VPS

---

## 14. AI Handoff Prompt (FINAL)

```
Continue from Reltroner ERP Phase 3 — Checkpoint #1.
Auth Gateway issues RMAT tokens and routes users to Finance module via /sso/consume.
Phase 2 SSO is fixed and frozen.
Phase 3 is partially complete; deployment and hardening not yet done.
```

---

## 15. One-Line Closing

**Checkpoint #1 marks that Phase 3 is architecturally correct, but operationally incomplete — the healthiest point to pause before advancing.**

---

**Author:** Reltroner
**System:** Reltroner ERP — Phase 3
**Checkpoint:** #1
**Status:** 🟡 In Progress
