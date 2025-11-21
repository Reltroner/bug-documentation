# 🛠️ Debugging Log — Fixing 403 Forbidden (Role Middleware Failure)

This document provides a **complete, technical, and reproducible** debugging record for resolving a **403 Forbidden** error encountered in the Reltroner HRM system. The issue originated from the custom `CheckRole` middleware and incorrect model relationships between `User`, `Employee`, and `Role`.

---

# 📌 Overview of the Issue

After logging into the application using Laravel Breeze authentication, accessing protected routes (e.g., `/dashboard`) consistently returned:

```
403 Forbidden
```

The log file (`storage/logs/laravel.log`) showed:

```
local.WARNING: Unauthorized access attempt (CheckRole)
{
  "user_id": 1,
  "current_role": null,
  "allowed_roles": ["Admin","HR Manager","Developer", ... ],
  "route": "dashboard"
}
```

The critical symptom:
`current_role` was **null**, meaning the role resolution pipeline failed.

---

# 🔍 Root Cause Summary

The root cause was a **broken relationship between `User` and `Employee`**:

* `users` table **contained** `employee_id = 11`
* There **was** an Employee record with `id = 11`
* But `$user->employee` returned **null**

This proved that the `User` model did **not** define the correct relationship (`belongsTo`), preventing the role from being resolved.

As a result:

* `CheckRole` middleware could not read `employee->role->title`
* No fallback role was found
* `current_role` remained null
* Middleware aborted with `403 Forbidden`

---

# 🧭 Full Debugging Process

## 1. Inspecting the Log

The first signal came from:

```
local.WARNING: Unauthorized access attempt (CheckRole)
"current_role": null
```

This confirmed the middleware was working, but the role resolution pipeline was failing.

---

## 2. Inspecting User Data via Tinker

```bash
php artisan tinker
```

```php
$user = \App\Models\User::find(1);
$user->employee; // returned null ✓ PROBLEM
$user->role;     // null
```

But the employee existed:

```php
\App\Models\Employee::find(11);
```

Returned:

```
id: 11
fullname: "Alice Admin"
role_id: 1
```

Therefore the model relationship was broken.

---

## 3. Confirming the Required Relationship

The schema showed:

### users table

```
employee_id BIGINT
```

### employees table

```
id BIGINT
role_id BIGINT
```

Thus:

```
User (employee_id) → Employee (id) → Role (id)
```

Meaning the correct relationship must be:

```php
// User.php
return $this->belongsTo(Employee::class, 'employee_id', 'id');
```

Not `hasOne`.

---

# 🏗️ Fix Implemented

## 1. Updated `User` model (complete and production-ready)

A fully upgraded model was created with:

* Correct `belongsTo` relationship
* Role resolution helper
* Display name accessor
* Optional support for Spatie Roles
* Consistent `$casts` and `$fillable` fields

(See full updated file in patch section below.)

---

## 2. Updated `Employee` model

```php
public function role()
{
    return $this->belongsTo(Role::class, 'role_id', 'id');
}
```

---

## 3. Cleaned up and upgraded `CheckRole` middleware

The middleware now:

* Normalizes allowed roles
* Resolves role from: session → employee → user → Spatie API
* Logs detailed context
* Avoids undefined variables
* Supports AJAX vs browser responses

---

## 4. Verified Via Tinker

After model updates:

```php
$user = User::with('employee.role')->find(1);

$user->employee;               // returns Employee object ✓
$user->employee->role->title;  // "Admin" ✓
$user->effectiveRole();        // "Admin" ✓
$user->hasRoleName('Admin');   // true ✓
```

---

## 5. Cleared Framework Caches

```bash
composer dump-autoload
php artisan config:clear
php artisan cache:clear
php artisan route:clear
php artisan view:clear
```

---

## 6. Final Verification

* Login as admin
* Access `/dashboard`
* No 403 error
* Log file contains no new warnings
* Middleware behaves correctly for both allowed and blocked roles

---

# ✔️ Final Result

The 403 Forbidden issue was **successfully resolved** by:

1. **Fixing the broken Eloquent relationships**
2. **Improving role resolution logic**
3. **Upgrading the `User` model**
4. **Hardening the `CheckRole` middleware**
5. **Validating using Tinker and Laravel logs**

All role-based routes now function correctly.

---

# 📂 Appendix – Final Updated User Model

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Support\Str;

class User extends Authenticatable
{
    use HasFactory, Notifiable;

    protected $fillable = [
        'name',
        'email',
        'password',
        'employee_id',
        'role',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected $casts = [
        'email_verified_at' => 'datetime',
        'password' => 'hashed',
    ];

    protected $appends = [
        'display_name',
    ];

    public function employee()
    {
        return $this->belongsTo(Employee::class, 'employee_id', 'id');
    }

    public function getDisplayNameAttribute(): string
    {
        return optional($this->employee)->fullname ?: $this->name;
    }

    public function effectiveRole(): ?string
    {
        $fromSession = session('role');
        if ($fromSession) return $fromSession;

        $fromEmployee = optional(optional($this->employee)->role)->title
            ?? optional($this->employee)->role;

        if ($fromEmployee) return $fromEmployee;

        return $this->role ?? null;
    }

    public function hasRoleName(string|array $role): bool
    {
        if (method_exists($this, 'hasRole')) {
            return $this->hasRole($role);
        }

        $effective = $this->effectiveRole();
        if (!$effective) return false;

        $allowed = is_array($role) ? $role : array_map('trim', explode(',', $role));
        $allowed = array_map(fn($r) => Str::lower($r), $allowed);

        return in_array(Str::lower($effective), $allowed, true);
    }
}
```

---

# 🧩 Lessons Learned

* Middleware failures often originate from **wrong model relationships**, not the middleware itself.
* Role logic must be resilient and allow multiple sources (session, employee, user).
* Always use Tinker to inspect real model states.
* Always check Laravel logs before changing code.
* Correctly using `belongsTo()` vs `hasOne()` is critical.

