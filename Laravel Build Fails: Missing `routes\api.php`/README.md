# 🐛 Laravel Build Fails: Missing `routes/api.php`

## ❗ Description

When deploying this Laravel project via Docker or CI/CD, the build process fails during the `composer install` or `artisan package:discover` phase with the following error:

```

ErrorException
require(/app/routes/api.php): Failed to open stream: No such file or directory

````

## 📍 Root Cause

Laravel's `RouteServiceProvider.php` includes this default route loading call:

```php
Route::middleware('api')
    ->prefix('api')
    ->group(base_path('routes/api.php'));
````

If the file `routes/api.php` does not exist, the deployment will break due to `require()` throwing an exception.

## ✅ Solution

To resolve this error, you must ensure the `routes/api.php` file exists.

### Option 1: Create Minimal API Route File

```php
// routes/api.php
<?php

use Illuminate\Support\Facades\Route;

Route::middleware('api')->get('/ping', function () {
    return response()->json(['status' => 'ok']);
});
```

This is the recommended solution and ensures the route group loads successfully.

### Option 2: Comment the API Route Loading (not recommended)

Edit `app/Providers/RouteServiceProvider.php` and comment out or remove the API route group if not needed:

```php
// Route::middleware('api')
//     ->prefix('api')
//     ->group(base_path('routes/api.php'));
```

> ⚠️ This will break any API usage in the project.

## 📌 Context

* Laravel version: 10 / 11 / 12
* Deployment target: Railway (Docker)
* Build step: `composer install` ➝ `artisan package:discover`

## 💡 Tip

Even if your project is not using API routes yet, it's best to keep a placeholder `routes/api.php` file to avoid deployment failure in the future.

---

**Status:** Fixed by creating the missing route file.
