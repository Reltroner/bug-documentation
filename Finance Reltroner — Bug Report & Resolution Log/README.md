# Finance Reltroner — Bug Report & Resolution Log

This document logs a debugging session related to the failure of hitting the endpoint:
```

GET /api/employees

```
...in a Laravel project structured as a modular ERP system. The `finance-reltroner` module was attempting to consume employee data through a gateway API (`gateway-reltroner`) which in turn fetched data from `hrm.reltroner.local`.

---

## 🐛 Bug Summary

### ❌ Browser Error
```

404 Not Found — [http://finance.reltroner.local:9002/api/employees](http://finance.reltroner.local:9002/api/employees)

```

### ✅ Laravel Console Output
```

2025-06-09 /api/employees ....................................................................... \~ 0.21ms

```

This suggests the route was **actually triggered**, but the browser returned 404, which made it appear as if the route didn’t exist.

---

## 🧠 Root Cause 1: `routes/api.php` Requires Prefix `/api`

Laravel automatically wraps all routes inside `routes/api.php` with a prefix of `/api`. Therefore, accessing:
```

[http://finance.reltroner.local:9002/employees](http://finance.reltroner.local:9002/employees)

````
...would return a 404 unless routed via `routes/web.php`.

### ✅ Resolution
Ensure your API route is declared with this structure:
```php
// routes/api.php
use App\Http\Controllers\API\EmployeeController;

Route::get('/employees', [EmployeeController::class, 'index']);
````

And that the request is made to:

```
http://finance.reltroner.local:9002/api/employees
```

---

## 🧠 Root Cause 2: APP\_URL Not Set to Correct Domain

If `APP_URL` in `.env` is still pointing to `localhost`, route helpers and service URL references may break or behave inconsistently during API requests.

### ✅ Resolution

Update `.env`:

```
APP_URL=http://finance.reltroner.local:9002
```

Then run:

```bash
php artisan config:clear
```

---

## 🧠 Root Cause 3: Cached Route Definitions

Laravel may cache routes or configs which can lead to unexpected 404s even when the route exists.

### ✅ Resolution

Run the following to flush all cached data:

```bash
php artisan route:clear
php artisan config:clear
php artisan cache:clear
```

---

## ✅ Extra Verification

Run:

```bash
php artisan route:list
```

Confirm that the route `/api/employees` exists with method `GET`.

---

## ✅ Final Status

After the fixes:

* The API route is now reachable at `http://finance.reltroner.local:9002/api/employees`
* The controller fetches employee data from: `http://gateway.reltroner.local:8000/hrm/employees`
* The full integration pipeline (Finance ← Gateway ← HRM) is working and returning JSON correctly.

---

## 🏁 Conclusion

This bug was caused by Laravel's route prefixing system combined with local misconfiguration during early-stage development. It was resolved through route inspection, environment tuning, and Laravel cache clearing.

---

Crafted with ❤️ by Rei Reltroner — 2025
