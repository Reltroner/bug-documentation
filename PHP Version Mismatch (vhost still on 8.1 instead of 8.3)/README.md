# 🐛 Bug Report — PHP Version Mismatch (vhost still on 8.1 instead of 8.3)

This document records the issue where the domain `hrm.reltroner.com` continued to run on **PHP 8.1.33** despite the project requiring **PHP >= 8.2**, and manual attempts to enforce PHP 8.3 via `.htaccess` did not take effect.

---

## 1. Problem Description

- Domain: `https://hrm.reltroner.com`
- Laravel project requires: **PHP ^8.2**
- CLI PHP 8.3 (`php83`) works correctly.
- However, accessing `_vcheck.php` via the browser still showed:

```

PHP\_VERSION=8.1.33

````

Even though `.htaccess` contained multiple `AddHandler` and `SetHandler` directives for PHP 8.3.

---

## 2. Root Cause

- The vhost for `hrm.reltroner.com` was still bound to **ea-php81**.
- Custom `.htaccess` handlers conflicted (multiple `AddHandler` / `SetHandler` lines).
- LiteSpeed / cPanel was ignoring the custom handler because the domain’s PHP version was not switched correctly in the control panel.

---

## 3. Solution Steps

### Option A — Use **MultiPHP Manager** (Recommended)

1. In **cPanel → MultiPHP Manager**:
   - Select `hrm.reltroner.com`.
   - Set **PHP Version** → `PHP 8.3 (ea-php83)`.
   - Click **Apply**.
   - Confirm that the PHP version column shows **ea-php83**.

2. Remove all custom handler lines from `.htaccess`:

```bash
for f in \
  ~/public_html/.htaccess \
  ~/www/.htaccess \
  ~/repositories/reltroner-hr-app/public/.htaccess
do
  [ -f "$f" ] && sed -i '/SetHandler .*php/d;/AddHandler .*php/d' "$f"
done
touch ~/public_html/lsphp_restart.txt
````

3. Re-check version:

```bash
echo '<?php header("Content-Type: text/plain"); echo "PHP_VERSION=", PHP_VERSION, "\n";' \
> ~/repositories/reltroner-hr-app/public/_vcheck.php
```

Open in browser:
`https://hrm.reltroner.com/_vcheck.php` → should now display **8.3.x**.

---

### Option B — If MultiPHP Does Not Apply (CloudLinux alt-php)

1. In **cPanel → Select PHP Version**:

   * Choose **PHP 8.3**.
   * Click **Set as Current**.

2. Refresh `_vcheck.php` (use Ctrl+F5 or Incognito to bypass cache).

---

### Option C — Fallback: Force Handler in `.htaccess`

If neither panel setting works, add this line at the top of `.htaccess` (above rewrites):

```apache
<IfModule mime_module>
    AddHandler application/x-httpd-alt-php83___lsphp .php
</IfModule>
```

Then:

```bash
touch ~/public_html/lsphp_restart.txt
```

Re-test `_vcheck.php`.

⚠️ **Important:** Do not mix `ea-php83` and `alt-php83` handlers simultaneously.

---

## 4. After PHP is Correctly Set

Once `_vcheck.php` shows **PHP 8.3.x**:

```bash
cd ~/repositories/reltroner-hr-app
php83 artisan config:clear
php83 artisan migrate --force --seed
```

Then open:

* `https://hrm.reltroner.com/`
* `https://hrm.reltroner.com/_vcheck.php`

---

## ✅ Resolution Status

* Cause: vhost still bound to PHP 8.1, ignoring Laravel’s requirement.
* Fix: Switch domain to PHP 8.3 via **MultiPHP Manager** (ea-php83) or **Select PHP Version** (alt-php83), and remove conflicting `.htaccess` handlers.
* Outcome: `_vcheck.php` shows **PHP 8.3.x**, Laravel application runs as expected.
