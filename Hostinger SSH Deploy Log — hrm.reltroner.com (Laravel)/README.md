# Hostinger SSH Deploy Log — `hrm.reltroner.com` (Laravel)

This README documents the exact steps and commands used to deploy **Reltroner HR App** to a Hostinger shared server via SSH. It also captures the error encountered (`destination path '.' already exists`) and the fix (clone into a temporary folder, then sync into the existing docroot).

> **Stack**: Laravel (PHP 8.3), Composer, cPanel/Hostinger, GitHub repo `Reltroner/reltroner-hr-app`
> **Docroot**: `~/domains/reltroner.com/public_html/hrm` (must serve from `public/`)

---

## 1) Context & Session Excerpt

**Local shell:**

```powershell
PS C:\laragon\www\reltronerhrapp> ssh -p 65002 u235364453@145.79.28.61
The authenticity of host '[145.79.28.61]:65002 ([145.79.28.61]:65002)' can't be established.
ED25519 key fingerprint is SHA256:i6WesTf1bhzsygbedCGk/dwdatOCZc1y2+Je+lwbc7Q.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[145.79.28.61]:65002' (ED25519) to the list of known hosts.
u235364453@145.79.28.61's password:
```

**Remote banner (Hostinger):**

```
Welcome back! The time now is 11:12 UTC
Server load: 7.11, 6.86, 6.88
Link to hPanel: https://hpanel.hostinger.com/
```

**Navigating to docroot & initial attempt:**

```bash
[u235364453@my-kul-web2046 ~]$ cd ~/domains/reltroner.com/public_html/hrm
[u235364453@my-kul-web2046 hrm]$ rm -f index.html
[u235364453@my-kul-web2046 hrm]$ git clone https://github.com/Reltroner/reltroner-hr-app .
fatal: destination path '.' already exists and is not an empty directory.
```

**Cause:** The `hrm` directory already contains files. Git refuses to clone into a **non-empty** directory.

---

## 2) Fix: Clone to a Temp Folder, Sync, Clean Up

Run these commands **from** `~/domains/reltroner.com/public_html/hrm`:

```bash
# 1) Clone the repo into a temporary folder (xsrc)
git clone https://github.com/Reltroner/reltroner-hr-app xsrc

# 2) Sync all contents (including dotfiles) into the current dir
# Prefer rsync; fall back to cp -a if rsync is unavailable
rsync -a xsrc/ ./  || cp -a xsrc/. ./

# 3) Remove the temporary folder
rm -rf xsrc

# 4) Sanity check the Laravel public directory exists
ls -la public
```

---

## 3) Install & Configure Laravel (Production)

```bash
# 5) Install PHP dependencies (optimize for production)
composer install --no-dev --optimize-autoloader

# 6) Create .env if missing, then edit it
[ -f .env ] || cp .env.example .env
nano .env
# Set at minimum:
# APP_NAME="Reltroner HRM"
# APP_ENV=production
# APP_DEBUG=false
# APP_URL=https://hrm.reltroner.com
# LOG_CHANNEL=stack
#
# DB_CONNECTION=mysql
# DB_HOST=localhost
# DB_PORT=3306
# DB_DATABASE=<hostinger_db_name>
# DB_USERNAME=<hostinger_db_user>
# DB_PASSWORD=<hostinger_db_pass>
#
# SESSION_DRIVER=file
# CACHE_STORE=file
# QUEUE_CONNECTION=sync
```

```bash
# 7) App key & database migrations
php artisan key:generate --force
php artisan migrate --force

# 8) Caches & file permissions
php artisan config:cache
php artisan route:clear      # avoid route:cache if you hit duplicate route issues
php artisan view:cache

chmod -R 775 storage bootstrap/cache
find public -type d -exec chmod 755 {} \;
find public -type f -exec chmod 644 {} \;

# 9) (Optional) Public storage for uploads
php artisan storage:link || true
```

> **Note:** On some shared hosts, `route:cache` can surface conflicts (e.g., duplicate `logout`). Use `route:clear` during initial bring-up; add `route:cache` later once routes are stable.

---

## 4) Web Server / Docroot Notes

* Ensure the subdomain **`hrm.reltroner.com`** docroot points to:

  ```
  ~/domains/reltroner.com/public_html/hrm/public
  ```

* Typical `.htaccess` (Laravel default) is already shipped in `/public/.htaccess`.

* If the host injects custom PHP handlers via `.htaccess`, remove conflicting `SetHandler`/`AddHandler` lines **outside** `public/` to avoid double handling.

---

## 5) Post-Deploy Checklist (hPanel)

1. **DNS** → *DNS Zone*: Create/confirm **A** record
   `hrm → 145.79.28.61`
2. **SSL** → *Security → SSL*: Issue/activate SSL for `hrm.reltroner.com`
3. **PHP Version**: PHP 8.3.x is recommended (matches development env)

---

## 6) Health Check

Quick probe page (remove after testing):

```bash
echo "<?php header('Content-Type: text/plain'); echo 'OK';" > public/health.php
```

Visit: `https://hrm.reltroner.com/health.php` → Expect `OK`
Then delete:

```bash
rm -f public/health.php
```

---

## 7) Troubleshooting

* **`destination path '.' already exists`**

  * You’re cloning into a non-empty directory. Use the **temp folder + rsync** method above.
* **500 errors after deploy**

  * Check `storage/logs/laravel.log`.
  * Ensure `storage/` and `bootstrap/cache/` permissions (see step 8).
  * Verify `.env` DB credentials & `APP_KEY`.
* **404 on all routes**

  * Confirm subdomain **docroot** points to `/public`.
  * Check `/public/.htaccess` exists and is readable.
* **Database migration errors**

  * Verify DB user privileges; create the DB beforehand in hPanel → *MySQL Databases*.
* **Mixed content / redirects**

  * Ensure `APP_URL=https://hrm.reltroner.com` and SSL is active.
* **Composer memory issues on shared hosting**

  * Try `COMPOSER_MEMORY_LIMIT=-1 composer install --no-dev --optimize-autoloader`.

---

## 8) Safe Re-deploy (Zero-downtime-ish on Shared Hosting)

```bash
# In docroot (hrm)
git clone https://github.com/Reltroner/reltroner-hr-app xsrc
rsync -a xsrc/ ./  || cp -a xsrc/. ./
rm -rf xsrc

composer install --no-dev --optimize-autoloader
php artisan migrate --force
php artisan config:cache && php artisan view:cache
```

---

## 9) Useful One-Liners

* **Who owns my files?**

  ```bash
  stat -c '%U:%G %n' storage bootstrap/cache | sort -u
  ```

* **Fix storage & cache perms (safe defaults):**

  ```bash
  chmod -R u=rwX,g=rwX,o=rX storage bootstrap/cache
  ```

* **Purge Laravel caches if behavior looks stale:**

  ```bash
  php artisan optimize:clear
  ```

---

## 10) Operational Notes

* This repo is intended to run **without Node** on shared hosting (Blade views). If you add Vite/asset builds, compile assets during CI and ship built files, or use a separate build host.
* Cron/queues: for background jobs, consider a lightweight schedule via hPanel **Cron Jobs** (e.g., `php artisan schedule:run` every minute). On basic plans, prefer `QUEUE_CONNECTION=sync`.

---

**Author:** Rei Reltroner (Raidan Malik Sandra)
**Environment:** Hostinger shared (my-kul-web2046), Asia/Jakarta (UTC+7)
**Last updated:** 2025-09-01
