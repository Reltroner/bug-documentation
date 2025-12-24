# 🧭 POST-MORTEM — Phase 2  
## Auth Gateway (Laravel) — Reltroner SSO

This document is a **complete, technical, and intentional post-mortem** of **Phase 2** in the Reltroner SSO project.  
It explains **what Phase 2 is**, **what it is NOT**, every **critical failure encountered**, the **correct fixes**, and the **core philosophy** that must never be violated.

This README exists to **prevent regression**, **misinterpretation**, and **over-engineering** in future phases.

---

## 🎯 Phase 2 Objective (MANDATORY CONTEXT)

**Phase 2 is NOT about:**
- ❌ RBAC
- ❌ User management
- ❌ Database design
- ❌ Production hardening
- ❌ Performance optimization

**Phase 2 exists ONLY to prove:**

> Laravel can authenticate **via Keycloak SSO**  
> → receive **authorization code**  
> → validate it  
> → create **server-side session**  
> → **without local login**  
> → **without database dependency**

Nothing more. Nothing less.

---

## 🧨 BUG #1 — Database Session Driver (The First Collapse)

### ❌ Error
```

SQLSTATE[HY000] [2002] No connection could be made
(Connection: mysql, SQL: select * from `sessions`)

```

### 🔍 Diagnosis
Laravel was configured with:
```

SESSION_DRIVER=database

```

But in Phase 2:
- MySQL was not running
- No database existed
- No `sessions` table existed

Laravel crashed **before**:
- routing
- controllers
- middleware
- SSO logic

### 🧠 Technical Meaning
Laravel tries to resolve session storage **on the very first request**.  
If the session driver is `database`, Laravel **requires a working DB immediately**.

### ✅ Correct Fix
```

SESSION_DRIVER=file

```

**Phase 2 does not require database sessions.**

---

## 🧨 BUG #2 — `DB_CONNECTION=null` (Logical Misconfiguration)

### ❌ Error
```

Undefined array key "driver"
(DatabaseManager.php:203)

```

### 🔍 Diagnosis
Environment config:
```

DB_CONNECTION=null

```

Laravel behavior:
- Always boots `DatabaseManager`
- Expects `config('database.connections.{driver}')`
- `null` is not a valid connection key

### 🧠 Technical Meaning
Laravel **never supports `DB_CONNECTION=null`**.  
Even if you "don’t use the database", Laravel still initializes DB services.

### ✅ Correct Fix
Use **SQLite in-memory**, not `null`:

```

DB_CONNECTION=sqlite
DB_DATABASE=:memory:

```

This provides:
- A valid DB driver
- Zero persistence
- Zero usage

---

## 🧨 BUG #3 — Cache Store = Database (Silent Time Bomb)

### ❌ Error
```

no such table: cache (Connection: sqlite)

```

Triggered during:
```

php artisan optimize:clear

```

### 🔍 Diagnosis
`.env` contained:
```

CACHE_STORE=database
CACHE_DRIVER=file

```

Laravel 10+ priority:
```

CACHE_STORE > CACHE_DRIVER

```

Result:
- Cache attempted to use database
- SQLite in-memory had no `cache` table
- `DELETE FROM cache` caused crash

### 🧠 Technical Meaning
Laravel cache commands **assume schema exists** if DB cache is enabled.

### ✅ Correct Fix (FINAL)
```

CACHE_STORE=file

```

And **remove entirely**:
```

CACHE_DRIVER
CACHE_STORE=database

```

**Phase 2 = file-based cache only.**

---

## 🧨 BUG #4 — HTTPS Requests to PHP Dev Server

### ❌ Terminal Error
```

Invalid request (Unsupported SSL request)

```

### ❌ Browser Error
```

ERR_CONNECTION_CLOSED

```

### 🔍 Diagnosis
- `php artisan serve` → **HTTP only**
- `.env` contained:
```

APP_URL=[https://app.reltroner.com](https://app.reltroner.com)

```

Browser behavior:
- Initiated HTTPS handshake
- PHP built-in server rejected it
- Connection closed immediately

### ✅ Correct Fix
```

APP_URL=[http://localhost:8000](http://localhost:8000)

```

---

## 🧨 BUG #5 — `APP_ENV=production` in Local Dev (FINAL DIAGNOSIS)

### ❌ Most Confusing Symptom
- Server log:
```

GET / ~ 0.45ms

```
- Browser:
```

ERR_CONNECTION_CLOSED

```

### 🔍 Final Diagnosis
```

APP_ENV=production

```

Laravel behavior in production:
- Stricter response handling
- Security-oriented output buffering
- Assumes hardened web server
- **Not compatible** with:
  - PHP built-in server
  - HTTP localhost
  - Dev workflows

Laravel **responded**, then **silently closed the connection**.

### ✅ FINAL FIX (LOCK PHASE 2)
```

APP_ENV=local
APP_DEBUG=true
APP_URL=[http://127.0.0.1:8000](http://127.0.0.1:8000)

````

### 📌 Why `127.0.0.1` (not `localhost`)
- Avoids HSTS cache
- Avoids forced HTTPS extensions
- Avoids proxy rewriting
- Avoids browser “helpfulness”

---

## 🧱 Final Configuration — Phase 2 (LOCKED)

```env
APP_ENV=local
APP_DEBUG=true
APP_URL=http://127.0.0.1:8000

DB_CONNECTION=sqlite
DB_DATABASE=:memory:

SESSION_DRIVER=file
CACHE_STORE=file
QUEUE_CONNECTION=sync
````

### Keycloak (Local)

```env
KEYCLOAK_BASE_URL=http://localhost:8080
KEYCLOAK_REALM=reltroner
KEYCLOAK_CLIENT_ID=reltroner-app
KEYCLOAK_CLIENT_SECRET=
KEYCLOAK_REDIRECT_URI=http://127.0.0.1:8000/callback
```

---

## 🧠 Phase 2 Core Philosophy (NON-NEGOTIABLE)

**Phase 2 is about proving flow, not stability.**

Therefore:

❌ Database = OFF
❌ Database cache = OFF
❌ HTTPS = OFF
❌ Production environment = OFF

✅ File-based everything
✅ HTTP only
✅ Local only
✅ Minimal moving parts

---

## 🧩 One-Line Final Summary

> **All Phase 2 bugs were caused by Laravel being configured too “production-minded” for local SSO flow testing — not by SSO itself.**

---

## 📎 Status

* Phase 2: **COMPLETE**
* Scope respected: **YES**
* Over-engineering avoided: **YES**
* Ready for Phase 3: **YES**
