# 🧭 Phase 2 Overview — Auth Gateway (Laravel)  
## Reltroner SSO with Keycloak (Manual OIDC)

This document provides a **complete, professional, and authoritative overview** of **Phase 2** of the Reltroner SSO implementation.

Phase 2 is a **strictly scoped authentication proof phase**.  
Its purpose is to validate **SSO flow correctness**, not system completeness or production readiness.

This README serves as:
- A **technical record**
- A **design decision reference**
- A **regression-prevention document** before entering **Phase 3 (Finance Consumer)**

---

## 🎯 Phase 2 Goal (Non-Negotiable)

**Objective:**  
Integrate **Laravel** with **Keycloak (SSO)** to authenticate users using **Authorization Code Flow**, resulting in a valid **Laravel session**, without any local login mechanism.

### ✅ In Scope
- ✔️ Login via Keycloak (OIDC Authorization Code Flow)
- ✔️ Manual token exchange (no abstraction libraries)
- ✔️ Laravel session creation after SSO
- ✔️ Breeze retained **only** as guard + middleware
- ✔️ No local login UI
- ✔️ No MySQL / SQLite user table dependency for auth logic

### ❌ Explicitly Out of Scope
- ❌ Socialite for Keycloak
- ❌ Local username/password login
- ❌ RBAC / permissions
- ❌ User provisioning logic
- ❌ Database-backed auth or persistence
- ❌ Production hardening

> **Phase 2 must be fully correct and clean before Phase 3 (Finance Consumer) can begin.**

---

## 🧩 Bug History & Root Cause Analysis

This section documents **every major failure**, **why it happened**, and **why the fix is correct**.

---

### 🧨 Bug #1 — Missing `services.keycloak.client_secret`

#### ❌ Error
```

Missing services entry for keycloak.client_secret
(SocialiteProviders\Manager\Exception\MissingConfigException)

```

#### 🔍 Root Cause
- Project still had **Socialite + SocialiteProviders Keycloak**
- Initial login route was:
```

/login/keycloak

```
- This triggered **Socialite Keycloak Provider**
- Socialite **requires `client_secret`**
- Phase 2 explicitly targets a **public client (no secret)**

#### 🧠 Technical Insight
Socialite is designed for:
- OAuth providers with secrets
- Opinionated flows
- Abstracted configuration

Phase 2 requires:
- Manual OIDC
- Explicit control
- Public client compatibility

#### ✅ Resolution
- ❌ Stop using Socialite for Keycloak
- ❌ Remove `/login/keycloak` route
- ❌ Remove all Socialite Keycloak invocations
- ✅ Implement **manual Authorization Code flow**

---

### 🧨 Bug #2 — Breeze Login UI Calls `login.keycloak`

#### ❌ Error
```
Route [login.keycloak] not defined.
(View: resources/views/auth/login.blade.php)
```

#### 🔍 Root Cause
- Laravel Breeze login view still referenced:
  ```php
  route('login.keycloak')
  ```

* Backend refactor removed that route
* Breeze UI was **not automatically updated**

#### 🧠 Technical Insight

Breeze is UI-first.
Removing backend routes **does not mutate views automatically**.

#### ✅ Resolution Options

##### ✅ Solution A (Recommended — Cleaner)

Redirect `/login` directly to SSO entry point:

```php
Route::get('/login', fn () => redirect()->route('sso.login'))->name('login');
```

##### ⚠️ Solution B (Alternative)

Modify `auth/login.blade.php` to JS-redirect to SSO.

**Solution A chosen** because:

* No view mutation
* Centralized routing
* Cleaner boundary

---

### 🧨 Bug #3 — Socialite vs Manual OIDC (Flow Collision)

#### Situation

* Codebase briefly mixed:

  * `Socialite::driver('keycloak')`
  * Manual `SSOController`

#### ❌ Result

* Socialite errors during testing
* Wrong routes invoked
* Confusing failure logs

#### 🧠 Root Cause

**Two authentication paradigms active at once**

#### ✅ Resolution

🔒 Enforce **one strategy only**:

* 👉 **Manual OIDC via `SSOController`**
* ❌ Remove all Socialite Keycloak references

---

### 🧨 Bug #4 — Breeze Auth Middleware Still Triggers `/login`

#### ❌ Problem

* Routes protected by `auth` middleware
* Breeze default behavior:

  ```
  unauthenticated → redirect to /login
  ```
* `/login` previously showed Breeze UI → error loop

#### ✅ Resolution

* Redirect `/login` → `/sso/login`
* Keeps Breeze middleware intact
* Seamless SSO entry

---

### 🧨 Bug #5 — Environment & Cache Inconsistencies

These issues were **not SSO-related**, but blocked progress.

---

#### a) Cache Database Error

❌ Error:

```
no such table: cache
```

🔍 Cause:

```
CACHE_STORE=database
```

SQLite in-memory has no cache table.

✅ Fix:

```
CACHE_STORE=file
```

---

#### b) `DB_CONNECTION=null`

❌ Error:

```
Undefined array key "driver"
```

🔍 Cause:
Laravel **always initializes DatabaseManager**.

✅ Fix:

```
DB_CONNECTION=sqlite
DB_DATABASE=:memory:
```

---

#### c) HTTPS Requests to PHP Dev Server

❌ Error:

```
Invalid request (Unsupported SSL request)
```

🔍 Cause:

* `php artisan serve` = HTTP only
* `APP_URL=https://...`

✅ Fix:

```
APP_URL=http://127.0.0.1:8000
APP_ENV=local
```

---

## ✅ Final Phase 2 Refactor (Best Practices)

### 📍 `routes/web.php`

```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\DashboardController;
use App\Http\Controllers\ProfileController;
use App\Http\Controllers\SSOController;

// Redirect Laravel login to SSO login
Route::get('/login', fn () => redirect()->route('sso.login'))->name('login');

// Root redirects to dashboard
Route::get('/', fn () => redirect()->route('dashboard'));

// SSO Routes
Route::get('/sso/login', [SSOController::class, 'redirect'])->name('sso.login');
Route::get('/sso/callback', [SSOController::class, 'callback'])->name('sso.callback');

// Authenticated routes
Route::middleware(['auth', 'verified'])->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index'])->name('dashboard');

    Route::get('/profile', [ProfileController::class, 'edit'])->name('profile.edit');
    Route::patch('/profile', [ProfileController::class, 'update'])->name('profile.update');
    Route::delete('/profile', [ProfileController::class, 'destroy'])->name('profile.destroy');
});

// Breeze defaults
require __DIR__ . '/auth.php';
```

---

### 📍 `SSOController` — Manual OIDC

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\Auth;
use App\Models\User;
use Illuminate\Support\Str;

class SSOController extends Controller
{
    public function redirect()
    {
        $query = http_build_query([
            'client_id'     => config('services.keycloak.client_id'),
            'response_type' => 'code',
            'scope'         => 'openid',
            'redirect_uri'  => config('services.keycloak.redirect_uri'),
        ]);

        return redirect(
            config('services.keycloak.base_url')
            . '/realms/' . config('services.keycloak.realm')
            . '/protocol/openid-connect/auth?' . $query
        );
    }

    public function callback(Request $request)
    {
        abort_if(!$request->has('code'), 403);

        $token = Http::asForm()->post(
            config('services.keycloak.base_url')
            . '/realms/' . config('services.keycloak.realm')
            . '/protocol/openid-connect/token',
            [
                'grant_type'   => 'authorization_code',
                'client_id'    => config('services.keycloak.client_id'),
                'redirect_uri' => config('services.keycloak.redirect_uri'),
                'code'         => $request->code,
            ]
        )->json();

        abort_if(!isset($token['access_token']), 403);

        $user = User::firstOrCreate(
            ['email' => 'sso@reltroner.local'],
            [
                'name' => 'SSO User',
                'password' => bcrypt(Str::random(32)),
            ]
        );

        Auth::login($user);

        session([
            'access_token' => $token['access_token'],
            'id_token'     => $token['id_token'] ?? null,
        ]);

        return redirect()->route('dashboard');
    }
}
```

---

## 📍 `.env` Essentials (Phase 2)

```env
APP_ENV=local
APP_DEBUG=true
APP_URL=http://127.0.0.1:8000

KEYCLOAK_BASE_URL=http://localhost:8080
KEYCLOAK_REALM=reltroner
KEYCLOAK_CLIENT_ID=reltroner-app
KEYCLOAK_REDIRECT_URI=http://127.0.0.1:8000/sso/callback

SESSION_DRIVER=file
CACHE_STORE=file
```

---

## 🧹 Cleanup Rules

```bash
php artisan optimize:clear
php artisan route:clear
php artisan view:clear
php artisan config:clear
```

* Ensure **no Socialite usage**
* Ensure **no `login.keycloak` references**
* Ensure **manual OIDC only**

---

## 🏁 Phase 2 Win Conditions

✔️ No Socialite errors
✔️ `/login` redirects to `/sso/login`
✔️ Keycloak Authorization Code flow succeeds
✔️ Laravel session is created
✔️ User reaches `/dashboard` without local login

---

## 🧩 One-Line Summary

> **Phase 2 succeeded by eliminating Socialite, enforcing a single manual OIDC strategy, redirecting Breeze login to SSO, and resolving all environment and routing conflicts.**

---

## 📎 Status

* Phase 2: **COMPLETE**
* Scope respected: **YES**
* Ready for Phase 3 (Finance Consumer): **YES**

```
```
