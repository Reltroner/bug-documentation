# 🛠 Fix: Undefined variable `$departments` on Dashboard View

This document records the fix applied to solve the error:

```

ErrorException: Undefined variable \$departments

````

## 📌 Problem Summary

While accessing the dashboard page at `/dashboard`, the Blade view `dashboard/index.blade.php` threw an error indicating that the variable `$departments` was not defined.

### Root Cause
The route definition for `/dashboard` was previously handled via a **closure** without passing any data to the view:

```php
Route::get('/dashboard', function () {
    return view('dashboard.index'); // ❌ no data passed
});
````

Since the view requires multiple data variables like `$departments`, `$employees`, `$payrolls`, etc., the view was not receiving them, causing the undefined variable error.

## ✅ Solution

We updated the route to use a controller that properly loads the data:

### `routes/web.php`

```php
use App\Http\Controllers\DashboardController;

Route::get('/dashboard', [DashboardController::class, 'index'])
    ->middleware(['auth', 'verified', 'role:Admin,HR Manager,Developer,Accountant,Data Entry'])
    ->name('dashboard');
```

### `app/Http/Controllers/DashboardController.php`

```php
public function index()
{
    $employees   = Employee::count();
    $departments = Department::count();
    $payrolls    = Payroll::count();
    $presences   = Presence::count();
    $tasks       = Task::all();

    return view('dashboard.index', compact(
        'employees', 'departments', 'payrolls', 'presences', 'tasks'
    ));
}
```

### `resources/views/dashboard/index.blade.php`

```blade
<h6 class="font-extrabold mb-0">{{ $departments }}</h6>
```

This view now works without error because the controller ensures the data is passed.

---

## 🔁 Lessons Learned

* Always ensure controllers are used when a view requires dynamic data.
* Avoid rendering Blade templates from route closures unless it’s a static or extremely simple page.
* Use the `compact()` helper or `with()` method to pass data to views consistently.

---

Let Astralis light the unknown ⚡
