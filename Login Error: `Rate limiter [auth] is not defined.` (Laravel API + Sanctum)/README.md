# Login Error: `Rate limiter [auth] is not defined.` (Laravel API + Sanctum)

This document explains a login failure caused by a missing **named rate limiter** in a Laravel API that uses Sanctum.

---

## ❗ Problem

**Request**
```http
POST http://127.0.0.1:8000/api/auth/login
Content-Type: application/json

{
  "email": "test@example.com",
  "password": "password",
  "device_name": "web"
}
````

**Response (500)**

```
Illuminate\Routing\Exceptions\MissingRateLimiterException
Rate limiter [auth] is not defined.
```

**Stack (excerpt)**

```
... ThrottleRequests.php
... MissingRateLimiterException::forLimiter('auth')
```

You may also see:

```
Class "App\Providers\RateLimiter" not found
```

if the necessary imports are missing in `AppServiceProvider`.

---

## 🔍 Root Cause

* Your route or middleware references a named throttle (e.g. `throttle:auth`), **but** no limiter named `auth` has been registered via `RateLimiter::for(...)`.
* Additionally, the provider is missing required imports for:

  * `Illuminate\Support\Facades\RateLimiter`
  * `Illuminate\Cache\RateLimiting\Limit`
  * `Illuminate\Http\Request`

---

## ✅ Fix

Define the named limiters in `App\Providers\AppServiceProvider` and import the correct classes.

**`app/Providers/AppServiceProvider.php`**

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;

// ✅ required imports
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;

// Optional (if you also customize reset/verify emails)
use Illuminate\Auth\Notifications\ResetPassword;
use Illuminate\Auth\Notifications\VerifyEmail;
use Illuminate\Notifications\Messages\MailMessage;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void {}

    public function boot(): void
    {
        // (Optional) Custom reset password URL for SPA/mobile
        ResetPassword::createUrlUsing(function ($user, string $token) {
            $base = config('app.frontend_reset_url', env('FRONTEND_RESET_URL', ''));
            if ($base) {
                return rtrim($base, '/')
                    . '?token=' . $token
                    . '&email=' . urlencode($user->getEmailForPasswordReset());
            }
            return url('/reset-password/'.$token.'?email='
                . urlencode($user->getEmailForPasswordReset()));
        });

        // (Optional) Custom verify email to SPA
        VerifyEmail::toMailUsing(function ($notifiable, string $url) {
            $base = config('app.frontend_verify_url', env('FRONTEND_VERIFY_URL', ''));
            if ($base) {
                $qs = parse_url($url, PHP_URL_QUERY);
                $spaUrl = rtrim($base, '/').($qs ? ('?'.$qs) : '');
                return (new MailMessage)
                    ->subject('Verify Email Address')
                    ->line('Click the button below to verify your email address.')
                    ->action('Verify Email', $spaUrl);
            }
            return (new MailMessage)
                ->subject('Verify Email Address')
                ->line('Click the button below to verify your email address.')
                ->action('Verify Email', $url);
        });

        // 🔒 Named limiter for login endpoints used by `throttle:auth`
        RateLimiter::for('auth', function (Request $request) {
            $key = strtolower($request->input('email', 'guest')) . '|' . $request->ip();
            return [ Limit::perMinute(10)->by($key) ];
        });

        // 🔒 Named limiter for password endpoints used by `throttle:password`
        RateLimiter::for('password', function (Request $request) {
            $key = strtolower($request->input('email', 'guest')) . '|' . $request->ip();
            return [ Limit::perMinute(5)->by($key) ];
        });
    }
}
```

Then clear caches and run the server:

```bash
php artisan optimize:clear
php artisan serve   # (note: the command is 'serve', not 'ser')
```

---

## 🧭 How to Use the Named Limiters

In your routes (e.g. `routes/api.php`), reference the names you defined:

```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\AuthController;
use App\Http\Controllers\PasswordController;

Route::post('/auth/login', [AuthController::class, 'login'])
    ->middleware('throttle:auth');

Route::post('/auth/forgot-password', [PasswordController::class, 'forgot'])
    ->middleware('throttle:password');

Route::post('/auth/reset-password', [PasswordController::class, 'reset'])
    ->middleware('throttle:password');
```

---

## 🧪 Test

```bash
curl -X POST http://127.0.0.1:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "password",
    "device_name": "web"
  }'
```

Expected: a JSON response with a Sanctum token (or 429 Too Many Requests if you exceed the defined limits).

---

## 🛠 Troubleshooting Checklist

* [ ] `AppServiceProvider@boot()` includes **imports** for `RateLimiter`, `Limit`, and `Request`.
* [ ] Named limiters (`auth`, `password`, etc.) are defined via `RateLimiter::for(...)`.
* [ ] Your routes actually reference the correct names (e.g. `throttle:auth`).
* [ ] Run `php artisan optimize:clear` after changes.
* [ ] No typos in Artisan commands (`serve`, not `ser` 😉).
* [ ] If using SPA/CSRF with Sanctum locally without HTTPS:

  ```
  SESSION_SECURE_COOKIE=false
  APP_URL=http://localhost
  ```

---

## 📝 Summary

* Error: `MissingRateLimiterException: Rate limiter [auth] is not defined.`
* Cause: Middleware references a named limiter that hasn’t been registered (and/or missing imports).
* Fix: Import the correct classes and define the limiter(s) in `AppServiceProvider::boot()`, then clear caches.
