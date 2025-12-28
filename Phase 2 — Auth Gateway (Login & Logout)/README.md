# 🔒 Phase 2 — Auth Gateway (Login & Logout)

**Project:** reltroner-app
**Role:** Authentication Gateway
**Identity Provider:** Keycloak
**Protocol:** OpenID Connect (Authorization Code Flow)
**Status:** ✅ **FROZEN (Completed & Stable)**

---

## 🎯 Objective

Phase 2 exists **only** to prove that **Laravel can operate as a pure Auth Gateway** backed by Keycloak.

This phase validates that Laravel can:

* Redirect users to Keycloak for authentication
* Receive an `authorization_code`
* Exchange code → tokens (`access_token`, `id_token`)
* Maintain **server-side session state**
* Perform **OIDC-compliant logout**
* Protect routes via middleware

🚫 **No local authentication is performed**
🚫 **No users are stored**
🚫 **No database is required**

---

## 🧱 System Boundary (Strict & Non-Negotiable)

### ❌ Explicitly Out of Scope (Phase 2)

Phase 2 **does NOT include**:

* User management
* User database / persistence
* RBAC or permissions
* Laravel Auth guards
* Socialite abstraction
* Production hardening
* Token persistence
* Multi-tenant logic

### ✅ Phase 2 Includes ONLY

* OIDC contract correctness
* Login flow correctness
* Logout flow correctness
* Session lifecycle correctness
* Gateway behavior correctness

---

## 🧠 Architecture Overview

```
Browser
   ↓
Laravel Gateway (reltroner-app)
   ↓ redirect
Keycloak (Authorization Server)
   ↓ callback (authorization_code)
Laravel Gateway
   ↓
Protected Routes (/dashboard)
```

**Key Principle**

* **Laravel = Gateway**
* **Keycloak = Source of Truth**

Laravel never authenticates credentials.
It only validates OIDC flows and manages sessions.

---

## 🔑 Login Flow (Final & Locked)

### 1️⃣ Redirect to Keycloak

**Endpoint**

```
GET /sso/login
```

**Authorization Request Parameters**

* `client_id`
* `response_type=code`
* `scope=openid`
* `redirect_uri` ⚠️ must be static & identical
* `state` (CSRF protection)

**Redirects to**

```
/realms/reltroner/protocol/openid-connect/auth
```

---

### 2️⃣ Callback from Keycloak

**Endpoint**

```
GET /sso/callback
```

**Validation Rules**

* `code` **must exist**
* `state` **must match session value**

Requests failing these checks are rejected immediately.

---

### 3️⃣ Token Exchange

**Endpoint**

```
POST /protocol/openid-connect/token
```

**Payload**

```
grant_type=authorization_code
client_id
redirect_uri  (MUST match authorization request)
code
```

**Returned Tokens**

* `access_token`
* `id_token`

---

### 4️⃣ Session Gateway Creation

Stored server-side session values:

```php
session([
    'sso_authenticated' => true,
    'access_token'      => '...',
    'id_token'          => '...',
]);
```

🚫 No user database
🚫 No `Auth::login()`
🚫 No guards

Session presence alone represents authentication state.

---

## 🛡️ Protected Routes

All protected routes use a **custom middleware**:

### `EnsureSSOAuthenticated`

**Logic**

```php
if (!session('sso_authenticated')) {
    return redirect()->route('sso.login');
}
```

This guarantees:

* No access without successful OIDC login
* Deterministic redirect behavior
* No reliance on Laravel auth internals

---

## 🚪 Logout Flow (Final & OIDC-Compliant)

### ⚠️ Critical Bug Encountered (Resolved)

**Initial mistake**

Using:

```
redirect_uri
```

**Result**

Keycloak rejected the request with:

```
Invalid parameter: redirect_uri
```

This error **never appeared in Laravel logs**, because Keycloak rejected the request **before redirecting back**.

---

### ✅ Correct OIDC Logout Strategy

Keycloak logout **requires**:

* ❌ `redirect_uri` → **NOT allowed**
* ✅ `post_logout_redirect_uri`
* ✅ `id_token_hint` (mandatory for public clients)

Logout **must redirect to a public endpoint**.

---

### 🔐 Final Logout Route (LOCKED)

```php
Route::get('/logout', function () {
    $idToken = session('id_token');

    session()->flush();

    return redirect()->away(
        rtrim(config('services.keycloak.base_url'), '/')
        . '/realms/' . config('services.keycloak.realm')
        . '/protocol/openid-connect/logout?'
        . http_build_query([
            'post_logout_redirect_uri' => 'http://localhost:8000/',
            'id_token_hint'            => $idToken,
        ])
    );
})->name('logout');
```

---

## 🔧 Keycloak Client Configuration (Final)

**Client ID**

```
reltroner-app
```

**Valid Redirect URIs**

```
http://localhost:8000/sso/callback
```

**Valid Post Logout Redirect URIs**

```
http://localhost:8000/
```

🚫 No wildcards
🚫 No `/dashboard`

---

## 🧠 Why `/dashboard` MUST NOT Be Used on Logout

* `/dashboard` is a **protected route**
* After logout → session is cleared
* Middleware redirects to `/sso/login`
* This can cause redirect loops or undefined behavior

Keycloak explicitly forbids protected post-logout landing pages.

➡️ **Logout must land on a public endpoint (`/`)**

---

## 🧪 Fully Verified Flow

### Login

```
/ → /sso/login → Keycloak → /sso/callback → /dashboard
```

### Logout

```
/dashboard → /logout → Keycloak logout → /
```

✔ Deterministic
✔ No loops
✔ No silent failures
✔ No ghost sessions

---

## 🧊 Phase 2 Status Matrix

| Component               | Status |
| ----------------------- | ------ |
| OIDC Login              | ✅      |
| Authorization Code Flow | ✅      |
| Token Exchange          | ✅      |
| Session Gateway         | ✅      |
| Middleware Protection   | ✅      |
| Logout (OIDC-Compliant) | ✅      |
| Database Dependency     | ❌      |
| User Model              | ❌      |

---

## 🚫 DO NOT TOUCH (PHASE 2 FROZEN)

The following are **explicitly forbidden** in Phase 2:

* ❌ Adding RBAC
* ❌ Adding user tables
* ❌ Introducing Socialite abstraction
* ❌ Changing logout parameters
* ❌ Modifying redirect URIs

Any of the above **breaks Phase 2 guarantees**.

---

## 🏁 Final Statement

Phase 2 intentionally stops **before complexity**.

This design mirrors:

* Enterprise SSO gateways
* Zero-trust entry points
* Modular ERP authentication layers

A senior engineer reviewing this repository should immediately recognize:

> This phase is **minimal, deterministic, and correct by design**.

---

**Author:** Reltroner
**Phase:** Auth Gateway — Phase 2
**Status:** 🔒 Frozen & Archived
