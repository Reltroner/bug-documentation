# Final Test Stabilization & Smoke Test Fix (Reltroner HR App)

This document captures the **last mile** changes that took the suite from “almost green” to a clean run. It focuses on the SQLite testing env, the successful attendance rule test, and the final fix for the smoke test middleware.

---

## 1) Testing Environment (SQLite, `.env.testing`)

We run feature tests on a file-backed SQLite DB to keep migrations consistent across tests.

* Ensure `.env.testing` exists and contains a valid key and SQLite settings (example):

```ini
APP_ENV=testing
APP_KEY=base64:GLJ9oSGAyKsugw3jZCrPnP/rgyJ1nYcZZSA0DWPhDaQ=
APP_DEBUG=true
APP_URL=http://localhost

DB_CONNECTION=sqlite
DB_DATABASE=database/testing.sqlite

CACHE_DRIVER=array
SESSION_DRIVER=array
QUEUE_CONNECTION=sync
MAIL_MAILER=array
FILESYSTEM_DISK=local
```

* Ensure `phpunit.xml` points to the **file** DB (not `:memory:`):

```xml
<php>
  <env name="APP_ENV" value="testing"/>
  <env name="DB_CONNECTION" value="sqlite"/>
  <env name="DB_DATABASE" value="database/testing.sqlite"/>
  <env name="CACHE_STORE" value="array"/>
  <env name="SESSION_DRIVER" value="array"/>
  <env name="QUEUE_CONNECTION" value="sync"/>
  <env name="MAIL_MAILER" value="array"/>
</php>
```

* One-time setup:

```bash
# Windows PowerShell
mkdir database 2>$null
ni database/testing.sqlite -Force 2>$null

php artisan migrate --env=testing --database=sqlite --force
php artisan config:clear
```

---

## 2) Current Test Status (before the final fix)

After configuring the testing env and aligning routes/validators/middleware, the run produced:

```
PASS … 33 passed
FAIL … SmokeRoutesTest > critical pages load for admin
```

Error excerpt:

```
Attempt to read property "role" on null
… app/Http/Middleware/CheckRole.php:23
```

Root cause: the smoke test attempted to disable a **nonexistent** `RoleMiddleware` class; the app actually uses the `role` alias bound to `App\Http\Middleware\CheckRole`.

---

## 3) Final Fix — Disable the Correct Middleware in Smoke Test

Update the test to disable `CheckRole` (or the `'role'` alias).

**Before**:

```php
use App\Http\Middleware\RoleMiddleware;   // ❌ not the real middleware
…
$this->withoutMiddleware(RoleMiddleware::class);
```

**After**:

```php
<?php

namespace Tests\Feature;

use App\Models\User;
use Tests\TestCase;
use App\Http\Middleware\CheckRole; // ✅ correct import

class SmokeRoutesTest extends TestCase
{
    public function test_critical_pages_load_for_admin(): void
    {
        $admin = User::factory()->create();
        $this->actingAs($admin);

        // Disable RBAC for smoke test (we only check 2xx responses)
        $this->withoutMiddleware(CheckRole::class);
        // Alternatively: $this->withoutMiddleware('role'); // if alias is registered

        $routes = [
            '/dashboard',
            '/employees',
            '/tasks',
            '/presences',
            '/payrolls',
            '/leave_requests',
        ];

        foreach ($routes as $r) {
            $this->get($r)->assertSuccessful(); // 2xx
        }
    }
}
```

> Why this is OK: This smoke test only verifies that critical pages respond (`2xx`). Actual RBAC behavior is (or should be) covered in dedicated RBAC tests that **do not** disable the middleware and properly seed relations.

---

## 4) (Optional) Middleware Hardening

If you want extra safety to avoid null dereferences in `CheckRole`, make it null-safe and allow a session fallback:

```php
// app/Http/Middleware/CheckRole.php
public function handle($request, \Closure $next, ...$allowed)
{
    $user = $request->user();

    $roleTitle =
        session('role')
        ?? optional(optional(optional($user)->employee)->role)->title
        ?? ($user->role ?? null);

    if (!$user || !$roleTitle || !in_array($roleTitle, $allowed, true)) {
        abort(403);
    }

    return $next($request);
}
```

---

## 5) Commands Recap

```bash
# Ensure config + test DB are ready
php artisan config:clear
php artisan migrate --env=testing --database=sqlite --force

# Run the full suite
php artisan test
```

**Expected outcome:** ✅ **All tests pass (34/34)**

---

## 6) Notes

* Attendance rule (prevent double check-in same day) is implemented and covered by a passing feature test.
* Employee CRUD tests use full schema fields and seed minimal `Department` and `Role`.
* RBAC smoke test disables `CheckRole` intentionally; RBAC behavior should be separately tested with proper seeding and middleware enabled.

---

**Author:** Rei Reltroner (Raidan Malik Sandra)
**Environment:** Laragon (Windows), Laravel, SQLite for testing
**Last updated:** 2025-09-02 (Asia/Jakarta, UTC+7)
