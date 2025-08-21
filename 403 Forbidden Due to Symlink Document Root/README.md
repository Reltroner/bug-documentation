# 🐛 Bug Report — 403 Forbidden Due to Symlink Document Root

This document records the issue where accessing the Laravel app resulted in **HTTP 403 Forbidden**, likely caused by the server blocking symlinked document roots (common on cPanel + LiteSpeed setups).

---

## 1. Problem Description

- Domain: `hrm.reltroner.com`
- Document root was set to a **symlink** → `~/public_html -> ~/repositories/reltroner-hr-app/public`.
- Result: Requests returned **403 Forbidden**, even though file permissions and `.htaccess` were correct.

---

## 2. Debugging Steps

### Step 1: Adjust `.htaccess` Options

Modified the first line:

```apache
Options +SymLinksIfOwnerMatch -Indexes
````

Then cleared config cache:

```bash
php83 artisan config:clear
```

Reloaded:

```
https://hrm.reltroner.com/index.php
```

➡️ Still **403 Forbidden** → confirms issue is not rewrite-related.

---

### Step 2: Test Static File Access

Created a simple HTML file:

```bash
echo "ok" > ~/repositories/reltroner-hr-app/public/health.html
```

Opened:

```
https://hrm.reltroner.com/health.html
```

➡️ Still returned **403** → confirms **symlinked docroot was blocked**.

---

## 3. Root Cause

* Many cPanel/LiteSpeed environments restrict symlinks as document roots for security.
* Even with correct permissions and `.htaccess`, requests fail with **403 Forbidden**.

---

## 4. Solution — Direct Docroot to `public` (Recommended)

Reconfigure the domain in cPanel to point directly to Laravel’s `public` folder instead of using a symlink.

1. Go to **cPanel → Domains → Manage** `hrm.reltroner.com` → **Remove Domain**.
2. **Re-create the domain**:

   * Domain: `hrm.reltroner.com`
   * **Document Root**:

     ```
     /**home/ynoqpang/repositories/reltroner-hr-app/public**
     ```
   * (Do not use `~/hrm.reltroner.com` or a symlink.)
3. (Optional) Run **AutoSSL** from **SSL/TLS Status**.
4. In **MultiPHP Manager**, ensure the domain uses **PHP 8.3**.
5. Test:

   * `https://hrm.reltroner.com`
   * `https://hrm.reltroner.com/index.php`

➡️ With docroot set directly to `public`, the symlink restriction no longer applies.
The **403 error should be resolved.**

---

## 5. Additional Troubleshooting (if needed)

* Temporarily disable **ModSecurity** for the domain via cPanel → **ModSecurity**.
* Check directory permissions:

```bash
ls -ld ~ ~/repositories ~/repositories/reltroner-hr-app ~/repositories/reltroner-hr-app/public
```

* Review server error logs via **cPanel → Errors**, since some hosts do not log in `~/logs`.

---

## ✅ Outcome

* Issue confirmed to be caused by **symlinked docroot restriction**.
* Solution: Point domain docroot directly to Laravel’s `public` directory.
* Once reconfigured, `403 Forbidden` should no longer occur.
