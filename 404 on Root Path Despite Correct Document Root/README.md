# 🐛 Bug Report — 404 on Root Path Despite Correct Document Root

This document records the issue where static files in `public/` load correctly, but accessing the domain root (`/`) still returned **404 Not Found**.

---

## 1. Problem Description

- Domain: `https://hrm.reltroner.com`
- Document root set to: `/home/ynoqpang/repositories/reltroner-hr-app/public`

Observed behavior:
- `https://hrm.reltroner.com/which.txt` → **404**
- `https://hrm.reltroner.com/test.txt` → ✅ `HELLO_FROM_PUBLIC`
- `https://hrm.reltroner.com/` → **404**, even though `index.php` exists in `public/`.

---

## 2. Root Cause Analysis

- Document root confirmed correct (static `test.txt` loads fine).
- The **index request (`/`) was not being rewritten to `index.php`**.
- Likely causes:
  1. `.htaccess` in `public/` not being read by LiteSpeed.
  2. Rewrite rules missing or misconfigured.
  3. PHP handler not applied for this subdomain.

---

## 3. Solution Steps

### Step 1 — Ensure `.htaccess` Exists and Contains Laravel Rules

File: `/home/ynoqpang/repositories/reltroner-hr-app/public/.htaccess`

Minimal required rules:

```apache
Options +FollowSymLinks -Indexes

<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteBase /

    # Redirect everything except existing files/folders to index.php
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]
</IfModule>
````

---

### Step 2 — Test Direct Access to `index.php`

Open:

```
https://hrm.reltroner.com/index.php
```

* If this works → issue is strictly `.htaccess` rewrite.
* If this also fails → PHP handler not active for the subdomain.

---

### Step 3 — Add PHP Handler for LiteSpeed

Append at the bottom of `.htaccess`:

```apache
<IfModule mime_module>
    AddHandler application/x-httpd-ea-php83___lsphp .php .php8 .phtml
</IfModule>
```

This ensures PHP 8.3 is used for the domain.

---

### Step 4 — Clear Laravel Cache

Run:

```bash
php83 artisan config:clear
php83 artisan route:clear
php83 artisan view:clear
php83 artisan cache:clear
```

---

## 4. Expected Outcome

* `https://hrm.reltroner.com/index.php` should display the Laravel front controller.
* `https://hrm.reltroner.com/` should be rewritten by `.htaccess` to `index.php`.
* Static files (e.g., `test.txt`) continue to load normally.

---

## ✅ Resolution Status

* Document root confirmed correct.
* Issue narrowed down to `.htaccess` rewrite or PHP handler configuration.
* Adding proper Laravel `.htaccess` rules and binding PHP 8.3 handler should resolve the **404 at root**.
