# Bug Documentation — Dashboard Route Resolution & Latent Fetch Error

## Scope
Finance Module — STEP 5.3 (READ Layer & Presentation Integration)

This document records and formalizes a routing and frontend integration bug discovered **after STEP 5.3 tests were fully passing**.  
The bug does **not violate accounting correctness**, but affects **navigation integrity and future maintainability**.

---

## 1. Bug Summary

Two related but distinct issues were identified:

1. **Route name mismatch (`dashboard` vs `dashboard.index`)**
2. **Latent frontend bug: fetching JSON from a View route**

Both issues surfaced during SSO flow and dashboard rendering, not during automated tests.

---

## 2. Bug #1 — Dashboard Route Name Mismatch

### Symptom

Runtime error during SSO redirect or view rendering:

```

Route [dashboard] not defined.
Route [dashboard.index] not defined.

````

### Observed Context

- `ConsumeController` redirects to:
  ```php
  redirect()->route('dashboard.index')
````

* Blade templates (`layouts/dashboard.blade.php`, `navigation.blade.php`) use:

  ```blade
  route('dashboard')
  ```

### Root Cause

The system inconsistently referenced **two different route names** for the same endpoint:

* `dashboard`
* `dashboard.index`

Additionally, attempts were made to define **two identical routes with different names**, which is not how Laravel route naming works.

### Incorrect Pattern (Observed)

```php
Route::get('/dashboard', ...)
    ->name('dashboard');

Route::get('/dashboard', ...)
    ->name('dashboard.index');
```

This does **not** create aliases.
The second definition overrides the first.

---

## 3. Bug #1 — Correct Resolution (Freeze-Safe)

### Canonical Decision

* **Single canonical route name**: `dashboard`
* All redirects and views must reference **the same name**

### Correct Route Definition

```php
Route::get('/dashboard', [DashboardController::class, 'index'])
    ->name('dashboard');
```

### Required Alignment

* `ConsumeController`:

  ```php
  return redirect()->route('dashboard');
  ```
* All Blade templates:

  ```blade
  route('dashboard')
  ```

### Why This Is STEP 5.3 Compliant

* No accounting logic touched
* No test contract altered
* Pure routing and presentation fix
* Does not affect read-model invariants

---

## 4. Bug #2 — Latent Frontend Fetch Error (Critical Insight)

### Symptom

No visible crash, but silent failure in JavaScript:

```js
fetch('/dashboard')
  .then(res => res.json())
```

### Root Cause

* `/dashboard` returns **HTML View**
* `res.json()` expects **JSON**
* Browser fails silently → charts never update

This bug is **latent**:

* Tests pass
* UI renders
* Error only appears in console (or not at all)

---

## 5. Architectural Violation Identified

| Layer        | Responsibility  | Status             |
| ------------ | --------------- | ------------------ |
| `/dashboard` | HTML View (SSR) | Correct            |
| JS Fetch     | JSON API        | ❌ Incorrect target |

**A View route must never be treated as an API endpoint.**

---

## 6. Correct Architectural Resolution (Non-Blocking)

### Existing Correct Endpoint

The system already provides a read-only internal endpoint:

```
GET /_internal/dashboard-summary
```

Defined as:

```php
Route::get('/_internal/dashboard-summary',
    [DashboardController::class, 'summary']
)->name('internal.dashboard.summary');
```

### Correct Fetch Usage

```js
fetch('/_internal/dashboard-summary')
  .then(res => res.json())
  .then(data => {
      // chart update logic
  });
```

### Alternative (Even Safer)

If charts are placeholders:

* **Remove the fetch entirely**
* Keep dashboard as pure SSR

---

## 7. Why Tests Did Not Catch This

| Area                  | Covered by Tests | Reason                |
| --------------------- | ---------------- | --------------------- |
| Accounting logic      | ✅ Yes            | STEP 5.3 tests        |
| Route name resolution | ❌ No             | No navigation tests   |
| JS fetch correctness  | ❌ No             | Frontend not asserted |
| SSO redirect target   | ❌ No             | Manual flow           |

This is expected and acceptable under current test scope.

---

## 8. Invariants Preserved

* Ledger immutability
* Trial Balance balance
* Financial statement correctness
* Read-only guarantees
* STEP 5.3 freeze doctrine

**No accounting contract was violated.**

---

## 9. Final Classification

| Aspect            | Classification                       |
| ----------------- | ------------------------------------ |
| Severity          | Medium (latent, not corrupting data) |
| Impact            | Navigation & frontend reliability    |
| Fix Type          | Routing + presentation               |
| Freeze Impact     | None                                 |
| Accounting Impact | None                                 |

---

## 10. Final Note

This bug highlights a critical architectural rule:

> **Views are not APIs.
> Route names are contracts.
> Silent failures are the most dangerous ones.**

Bug resolved without breaching STEP 5.3 freeze.

```
```
