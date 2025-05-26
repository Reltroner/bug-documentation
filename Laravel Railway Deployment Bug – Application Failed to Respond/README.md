# 🐛 Laravel Railway Deployment Bug – Application Failed to Respond

## 📋 Description

During deployment of the Laravel-based `reltroner-hr-app` on [Railway](https://railway.app), the application failed to respond with a 502 error, even though all migrations and environment configurations were successfully processed.

The application logs showed:

```

Server running on \[[http://0.0.0.0:9000](http://0.0.0.0:9000)].

````

However, the deployed site at `https://hrm.reltroner.com` or `https://reltroner-hrm.up.railway.app` remained inaccessible with the error:

> Application failed to respond  
> ERR_ADDRESS_INVALID or 502 Bad Gateway

## 📍 Root Cause

Using the following command in Railway's **Start Command** caused the error:

```bash
php artisan serve --host=0.0.0.0 --port=9000
````

This command runs Laravel's built-in development server, which is **not intended for production use** and **does not bind correctly in Railway's containerized environment**.

## ✅ Final Fix

Replace the **Start Command** in Railway's deployment settings with:

```bash
php -S 0.0.0.0:$PORT -t public
```

### ✅ Why it works:

* Uses PHP’s built-in server (production-safe for simple hosting)
* `$PORT` is dynamically assigned by Railway
* `-t public` ensures Laravel routes are correctly served from `/public/index.php`

## 📦 Optional (if using Nixpacks or Docker)

You can alternatively use a `Procfile` or Dockerfile for more advanced setups:

```Procfile
web: php -S 0.0.0.0:$PORT -t public
```

## 🧪 Confirmed Working

After applying the fix, the application responded properly with:

* ✅ Working root route at `hrm.reltroner.com`
* ✅ Database seeded and migrated
* ✅ No more 502 or address binding issues

---

## 💡 Pro Tip

Never use `php artisan serve` in production — it's intended for local dev only.

---

### 🔗 Related Docs

* [Railway PHP Deployment Guide](https://docs.railway.app/guides/php)
* [Laravel Serve Docs](https://laravel.com/docs/11.x/valet#the-serve-command)

---

✅ Issue resolved as of `May 26, 2025 @ 7:30 PM`

