# ⚠️ Fixing Logout and "Form is not secure" Issues in Laravel + Keycloak (Railway Deployment)

This guide documents how to resolve two common issues encountered when using Laravel with Keycloak SSO, deployed on Railway:

---

## ✅ 1. "Form is not secure" Warning

### ❌ Problem

Submitting forms on routes like `/logout` or `/profile` triggers a browser warning:

> *"The information you're about to submit is not secure."*

This is because the form is being submitted over HTTP instead of HTTPS.

---

### ✅ Solution: Enforce HTTPS Everywhere

#### 🔧 Step 1: Create a Middleware to Force HTTPS

```bash
php artisan make:middleware ForceHttps
````

Edit `app/Http/Middleware/ForceHttps.php`:

```php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class ForceHttps
{
    public function handle(Request $request, Closure $next)
    {
        if (!$request->secure() && app()->environment('production')) {
            return redirect()->secure($request->getRequestUri());
        }

        return $next($request);
    }
}
```

---

#### 🔧 Step 2: Register Middleware Globally

In `app/Http/Kernel.php`, register it in the `$middleware` array:

```php
protected $middleware = [
    \App\Http\Middleware\ForceHttps::class,
    \App\Http\Middleware\BlockBotUserAgent::class,
];
```

---

#### 🔧 Step 3: Set `APP_URL` to HTTPS in `.env`

```env
APP_URL=https://app.reltroner.com
```

---

## ✅ 2. Laravel Logout Route Conflict

### ❌ Problem

Laravel Jetstream or Fortify already registers a `logout` route with the name `logout`. Defining your own route with the same name causes conflicts during route\:cache.

---

### ✅ Solution: Rename Custom Logout Route

Update your logout route in `routes/web.php`:

```php
Route::get('/logout', function () {
    Auth::logout();

    $keycloakLogoutUrl = env('KEYCLOAK_LOGOUT_URL', 'https://sso.reltroner.com/realms/reltroner/protocol/openid-connect/logout');
    $redirectUri = urlencode(config('app.url') . '/login/keycloak');

    return redirect("$keycloakLogoutUrl?redirect_uri=$redirectUri");
})->name('keycloak.logout'); // ✅ Avoid conflict with Laravel's default logout route
```

Update your Blade sidebar/menu:

```blade
<form method="POST" action="{{ route('keycloak.logout') }}">
    @csrf
    <button type="submit" class="sidebar-link btn btn-link text-start w-100 p-0 m-0">
        <i class="bi bi-box-arrow-right"></i>
        <span>Logout</span>
    </button>
</form>
```

> You can also simplify to `<a href="{{ route('keycloak.logout') }}">Logout</a>` if CSRF protection is not needed for logout.

---

## 🔒 Bonus: Force HTTPS URLs at Global Level

If Railway (or any proxy) doesn’t correctly handle HTTPS detection, Laravel might generate HTTP URLs internally.

In `App\Providers\AppServiceProvider.php`, inside `boot()` method:

```php
if (app()->environment('production')) {
    \URL::forceScheme('https');
}
```

---

## ✅ Results After Fixing

* 🔐 No more “Form is not secure” warnings
* 🔁 Logout redirects to Keycloak login securely
* 🚫 No route name conflict (`logout`)
* ☁️ Fully production-ready with HTTPS enforced

---

## 🛠 Need Help?

If you're still encountering issues, feel free to check:

* `config/session.php`
* `config/services.php`

Or open an issue with reproduction steps.
