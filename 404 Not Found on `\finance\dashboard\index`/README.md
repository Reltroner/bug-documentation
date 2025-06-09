# 🐞 Bug Report: 404 Not Found on `/finance/dashboard/index`

## ❓ Problem

While trying to access the Finance dashboard at:

```

[http://finance.reltroner.local:9002/finance/dashboard/index](http://finance.reltroner.local:9002/finance/dashboard/index)

```

The browser returns:

```

404 Not Found

````

Even though the route was correctly defined inside the `routes/web.php` of the `finance-reltroner` Laravel project.

---

## ⚙️ System Setup

- Laravel app: `finance-reltroner`
- Route file: `routes/web.php`
- Laravel dev server command:

```bash
php artisan serve --host=finance.reltroner.local --port=9002
````

* Windows `hosts` file entry:

```txt
127.0.0.1 finance.reltroner.local
```

---

## 🔍 Investigation Summary

1. **Correct Route Exists:**

   ```php
   // routes/web.php
   Route::get('/finance/dashboard/index', [DashboardController::class, 'index'])->name('finance.dashboard');
   ```

2. **But Browser Access Used Invalid Domain:**

   ```txt
   http://reltroner.test/finance/dashboard/index ❌ WRONG
   http://finance.reltroner.local:9002/finance/dashboard/index ✅ CORRECT
   ```

   * The Laravel development server is not registered under `reltroner.test` domain.
   * The `php artisan serve` command only binds to `finance.reltroner.local`.

3. **DNS\_PROBE\_POSSIBLE / 404 Occurs** due to either:

   * Missing entry in `hosts` file.
   * Accessing wrong domain not tied to that Laravel app.

---

## ✅ Solution

Always access the Finance dashboard through the correct domain and port that matches the development server.

```txt
✅ Access this:
http://finance.reltroner.local:9002/finance/dashboard/index
```

```bash
php artisan serve --host=finance.reltroner.local --port=9002
```

---

## ✅ Optional: Want to Use `reltroner.test/finance/...`?

You must:

* Create a unified Laravel application (gateway).
* Delegate rendering from submodules (finance, hrm, etc.) using subfolder routing.
* Mount submodule views like `resources/views/finance/...`
* Declare centralized routes in `reltroner-app-main`

This architecture is suitable only for a **monolith** Laravel ERP, not for modular microservices.

---

## 📎 Related Context

* Laravel route list confirmed via `php artisan route:list`
* Hosts file entries correctly include:

```txt
127.0.0.1 finance.reltroner.local
127.0.0.1 reltroner.test
```

* Frontend fetch URL that failed initially:

```js
fetch('/finance/dashboard')
```

Expected to be relative, but domain mismatch caused the 404.

---

## 🧠 Final Suggestion

Stick to using subdomain-based architecture (`*.reltroner.local`) for modular Laravel apps.

If a unified gateway is needed later under `reltroner.test`, restructure routing and integrate modules as packages or subfolders.

---

Let Astralis light the unknown. 🌌
