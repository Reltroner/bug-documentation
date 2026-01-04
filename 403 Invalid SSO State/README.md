# 🐞 Bug Report — 403 Invalid SSO State

**Keycloak ↔ Laravel Gateway (Reltroner Auth Gateway — Phase 2)**

---

## 📌 Summary

**Bug**
Users intermittently received a **`403 Invalid SSO state`** error **after a successful login in Keycloak**, during the redirect back to the Gateway (`/sso/callback`).

**Impact**
Users were unable to complete the SSO login process, effectively blocking access to the entire ERP ecosystem.

**Status**
✅ **FIXED**

---

## 🌍 Environment

| Component      | Value                 |
| -------------- | --------------------- |
| Gateway        | `app.reltroner.com`   |
| Auth Provider  | Keycloak              |
| Framework      | Laravel               |
| Session Driver | `file`                |
| Environment    | Local                 |
| Phase          | Phase 2 (SSO Gateway) |
| Phase 3 Impact | ❌ No direct impact    |

---

## ⏱ Timeline of Events

1. User opens `http://localhost:8000`

2. Gateway redirects to `/sso/login`

3. Gateway redirects to **Keycloak Authorization Endpoint**

4. User successfully authenticates in Keycloak

5. Keycloak redirects back to:

   ```
   http://localhost:8000/sso/callback?code=...&state=...
   ```

6. Gateway immediately responds with:

   ```
   403
   Invalid SSO state
   ```

7. No error appears in `storage/logs/laravel.log`

---

## 🚨 Symptoms

* ❌ Login fails **after** successful Keycloak authentication
* ❌ Error occurs **before token exchange**
* ❌ No Laravel log entries
* ❌ Always returns HTTP 403
* ❌ Occurs **intermittently** (non-deterministic)

---

## 📍 Affected Code Location

**File**

```
app/Http/Controllers/SSOController.php
```

**Critical Logic**

```php
abort_if(
    !$request->has('state') || $request->state !== session('sso_state'),
    403,
    'Invalid SSO state.'
);
```

---

## 🔍 Root Cause Analysis (RCA)

### Primary Root Cause

> **The session was not stable or persisted between the redirect to Keycloak and the callback back to the Gateway.**

Specifically:

* `sso_state` was stored in the session **before redirect**
* The session was **not regenerated**
* As a result:

  * Session ID changed
  * Previous session data was lost
  * `session('sso_state')` returned `null` during callback
* State validation failed → `403 Invalid SSO state`

---

### Technical Details

#### ❌ Before Fix

```php
$state = bin2hex(random_bytes(16));
session(['sso_state' => $state]);
```

**Problem**

* Session could be:

  * regenerated implicitly by Laravel
  * overwritten by middleware
  * not fully persisted to storage yet

This made the `state` value unreliable across redirects.

---

## 🧠 Why There Was No Laravel Log

* The failure was triggered via `abort_if()`
* This is **not an exception**
* It does **not pass through try/catch**
* Therefore:

  * No stack trace
  * No entry in `laravel.log`

This behavior is expected and correct.

---

## ❌ Factors That Were NOT the Cause

The following were **explicitly ruled out**:

* ❌ Keycloak misconfiguration
* ❌ Redirect URI mismatch
* ❌ Invalid or expired token
* ❌ Phase 3 RMAT logic
* ❌ Cookie SameSite issues
* ❌ HTTPS vs HTTP mismatch

None of these contributed to the bug.

---

## 📉 System Impact

| Area     | Impact            |
| -------- | ----------------- |
| Login    | ❌ Failed          |
| Gateway  | ❌ Unusable        |
| Finance  | ❌ Inaccessible    |
| Security | ✅ No data leakage |
| Data     | ✅ Safe            |

---

## 🛠 Final Fix (Required)

### 🔧 Code Change

**File**
`app/Http/Controllers/SSOController.php`

**Method**
`redirect()`

---

### ✅ After Fix

```php
public function redirect(Request $request)
{
    // 🔒 Stabilize session before OIDC flow
    $request->session()->regenerate();

    $redirectUri = rtrim(config('services.keycloak.redirect_uri'), '/');

    $state = bin2hex(random_bytes(16));
    session(['sso_state' => $state]);

    $query = http_build_query([
        'client_id'     => config('services.keycloak.client_id'),
        'response_type' => 'code',
        'scope'         => 'openid',
        'redirect_uri'  => $redirectUri,
        'state'         => $state,
    ]);

    return redirect()->away(
        rtrim(config('services.keycloak.base_url'), '/')
        . '/realms/' . config('services.keycloak.realm')
        . '/protocol/openid-connect/auth?' . $query
    );
}
```

### 🔑 Key Fix

```php
$request->session()->regenerate();
```

This guarantees session persistence **before** entering the OIDC redirect flow.

---

## ✅ Fix Verification

### Manual Test

1. Open browser in incognito mode
2. Navigate to `http://localhost:8000`
3. Login via Keycloak
4. Confirm:

   * ❌ No 403 error
   * ✅ Successful access to Gateway dashboard

---

### Negative Testing

* Tamper with `state` parameter → correctly returns 403
* Delete session cookie → request rejected securely

Security behavior remains intact.

---

## 🔄 Impact on Phase 3

| Aspect                 | Status           |
| ---------------------- | ---------------- |
| RMAT                   | ❌ Not affected   |
| Gateway Token Handling | ❌ Not affected   |
| Finance `/sso/consume` | ❌ Not affected   |
| Phase 3 Architecture   | ✅ Clean & stable |

---

## 🛡 Preventive Measures

1. **Always regenerate session before OIDC redirects**
2. Add lightweight logging for:

   * session ID
   * hashed `state` value (never raw)
3. Never:

   * remove `state` validation
   * disable CSRF protection
4. Keep domain usage consistent (`localhost` vs `127.0.0.1`)

---

## 📘 Lesson Learned

> **OIDC `state` is not just a parameter—it is entirely dependent on session stability.
> Without proper session regeneration, correct security behavior can appear as a bug.**

---

## 🟢 Final Status

* 🟢 Bug resolved
* 🟢 Security preserved
* 🟢 Phase 2 stable
* 🟢 Phase 3 unaffected

---

**Author:** Reltroner
**System:** Auth Gateway — Phase 2
**Status:** ✅ Resolved & Documented

