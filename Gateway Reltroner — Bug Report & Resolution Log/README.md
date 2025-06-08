# Gateway Reltroner — Bug Report & Resolution Log

This document contains a detailed log of an integration issue between `gateway-reltroner` and `hrm.reltroner.local`, encountered during local development under Laragon. It includes diagnosis, causes, and resolutions for transparency and future debugging.

---

## 🐛 Bug Report: "Failed to fetch from HRM"

### ❌ Error Message (Browser Output)
```json
{"error": "Failed to fetch from HRM"}
````

### 🧱 Error Stack Trace (Laravel)

```
Illuminate\Http\Client\ConnectionException
cURL error 7: Failed to connect to hrm.reltroner.local port 80
```

---

## 🧠 Root Cause #1: DNS Not Found

### ✅ Resolution Steps

* `hrm.reltroner.local` was not registered in the `hosts` file.
* Fixed by editing: `C:\Windows\System32\drivers\etc\hosts`

```txt
127.0.0.1   hrm.reltroner.local
127.0.0.1   gateway.reltroner.local
```

* Rebooted Laragon or re-ran Laravel dev servers.

---

## 🧠 Root Cause #2: Target App Not Running

Even with proper DNS, `hrm.reltroner.local` was unreachable because no Laravel dev server was running under that hostname.

### ✅ Resolution Steps

```bash
cd C:/laragon/www/reltronerhrapp
php artisan serve --host=hrm.reltroner.local --port=80
```

---

## 🧠 Root Cause #3: Invalid Column in Query

After resolving the network layer, the following SQL exception occurred:

```
SQLSTATE[42S22]: Column not found: 1054 Unknown column 'position' in 'field list'
```

### ❌ Problematic Code in HRM

```php
Employee::select('id', 'fullname', 'email', 'position', 'department_id')->get();
```

### ✅ Resolution

Column `position` does not exist in the `employees` table. It was removed and replaced with available fields.

### ✔️ Final Fixed Route in HRM (`routes/web.php`)

```php
use Illuminate\Support\Facades\Response;
use App\Models\Employee;

Route::get('/api/public-employees', function () {
    return Response::json(
        Employee::select('id', 'fullname', 'email', 'department_id', 'role_id')->get()
    );
});
```

---

## ✅ Gateway Route That Works

```php
Route::get('/hrm/employees', function () {
    $hrmUrl = config('services.modules.hrm') . '/api/public-employees';

    $response = Http::get($hrmUrl);

    if ($response->successful()) {
        return response()->json($response->json());
    }

    return response()->json(['error' => 'Failed to fetch from HRM'], $response->status());
});
```

---

## ✅ Result (JSON Output)

```json
[
  {
    "id": 1,
    "fullname": "Marsha Rainwell",
    "email": "marsha@reltroner.com",
    "department_id": 2,
    "role_id": 1
  }
]
```

---

## 🏁 Status: **Resolved & Working**

Integration between `gateway-reltroner` and `hrm.reltroner.local` is now functional and stable in development mode.

---

Crafted with ❤️ by Rei Reltroner — 2025
