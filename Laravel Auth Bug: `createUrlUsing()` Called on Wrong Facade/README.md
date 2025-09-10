# Laravel Auth Bug: `createUrlUsing()` Called on Wrong Facade

This repository documents a bug encountered when customizing **password reset URLs** in Laravel.

---

## ⚠️ Problem

Running:

```bash
php artisan optimize:clear
````

Resulted in:

```
Error 

Call to undefined method Illuminate\Auth\Passwords\PasswordBroker::createUrlUsing()

at vendor\laravel\framework\src\Illuminate\Auth\Passwords\PasswordBrokerManager.php:152
```

This happened because the code in `AppServiceProvider` incorrectly used:

```php
Password::createUrlUsing(...)
```

But the **`createUrlUsing()`** method does **not** exist on the `Password` facade.
Instead, it belongs to `Illuminate\Auth\Notifications\ResetPassword`.

---

## ✅ Solution

Update `app/Providers/AppServiceProvider.php` to use the correct class.

### Before (❌ Wrong)

```php
use Illuminate\Support\Facades\Password;

Password::createUrlUsing(function ($user, string $token) {
    // ...
});
```

### After (✅ Fixed)

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Auth\Notifications\ResetPassword;   // ✅ Correct import
use Illuminate\Auth\Notifications\VerifyEmail;     // (optional)
use Illuminate\Notifications\Messages\MailMessage; // (optional)

class AppServiceProvider extends ServiceProvider
{
    public function register(): void {}

    public function boot(): void
    {
        // ✅ Custom reset password URL (for SPA/mobile frontend)
        ResetPassword::createUrlUsing(function ($user, string $token) {
            $base = config('app.frontend_reset_url', env('FRONTEND_RESET_URL', ''));
            if ($base) {
                return rtrim($base, '/')
                    . '?token=' . $token
                    . '&email=' . urlencode($user->getEmailForPasswordReset());
            }

            // fallback to default Laravel route
            return url('/reset-password/'.$token.'?email='
                . urlencode($user->getEmailForPasswordReset()));
        });

        // (Optional) Custom verify email notification
        VerifyEmail::toMailUsing(function ($notifiable, string $url) {
            $base = config('app.frontend_verify_url', env('FRONTEND_VERIFY_URL', ''));
            if ($base) {
                $qs = parse_url($url, PHP_URL_QUERY);
                $spaUrl = rtrim($base, '/').($qs ? ('?'.$qs) : '');
                return (new MailMessage)
                    ->subject('Verify Email Address')
                    ->line('Click the button below to verify your email address.')
                    .action('Verify Email', $spaUrl);
            }

            return (new MailMessage)
                ->subject('Verify Email Address')
                ->line('Click the button below to verify your email address.')
                ->action('Verify Email', $url);
        });
    }
}
```

---

## 🔧 Steps to Fix

1. Remove `use Illuminate\Support\Facades\Password;`
2. Import `use Illuminate\Auth\Notifications\ResetPassword;`
3. Run:

```bash
php artisan optimize:clear
php artisan serve
```

---

## 🌐 Local Dev Notes

If using **Laravel Sanctum** and CSRF in a local (non-HTTPS) environment, set in `.env`:

```
SESSION_SECURE_COOKIE=false
APP_URL=http://localhost
```

Otherwise, cookies may not work properly.

---

## 📝 Summary

* The error occurs because `createUrlUsing()` does **not** exist on the `Password` facade.
* Correct usage is via `ResetPassword::createUrlUsing(...)`.
* After fixing `AppServiceProvider`, password reset links for SPA/mobile frontends work as expected.

---

### References

* [Laravel Docs – Password Reset](https://laravel.com/docs/passwords)
* [Laravel Docs – Notifications](https://laravel.com/docs/notifications)
