# README — Seeding on Production (Faker error) and Finalize Deploy

This doc records exactly what happened while deploying **Reltroner HR App** on Hostinger and how the production database was seeded safely without leaving dev packages behind.

> **Stack**: Laravel • PHP 8.3 • Composer • Hostinger (shared)
> **Dir**: `~/domains/reltroner.com/public_html/hrm`
> **URL**: `https://hrm.reltroner.com` (docroot must point to `/public`)

---

## 1) What happened

You ran migrations and then attempted to refresh + seed:

```bash
php artisan migrate --force
# ...
php artisan migrate:fresh --seed --force
```

During seeding, Laravel threw:

```
Class "Faker\Factory" not found
```

### Why it happened

* **Faker** lives in `require-dev` (dev dependencies).
* On the server you previously installed with `--no-dev`, so **Faker wasn’t present**.

---

## 2) The fix (recommended approach used)

Keep production lean, but temporarily install dev packages **only to seed**, then return to `--no-dev`.

```bash
# A) Install with dev packages (temporarily)
php composer.phar install

# B) Seed the database
php artisan db:seed --force

# C) Switch back to production mode (no dev deps)
php composer.phar install --no-dev --optimize-autoloader

# D) Rebuild caches
php artisan config:cache
php artisan route:clear
php artisan view:cache
```

This keeps Faker out of the final production image while allowing seeding to complete.

> Alternative (not used): `php composer.phar require fakerphp/faker:^1.23` to keep Faker installed in prod permanently. Simpler, but increases prod surface area.

---

## 3) Full console log (key excerpts)

**Migrations OK:**

```
php artisan migrate --force

INFO  Preparing database.
Creating migration table ... DONE
INFO  Running migrations.
0001_01_01_000000_create_users_table ................ DONE
0001_01_01_000001_create_cache_table ............... DONE
0001_01_01_000002_create_jobs_table ................ DONE
2025_05_12_083807_create_human_resources_app ....... DONE
2025_05_17_090430_alter_tabel_user ................. DONE
2025_05_21_133358_update_checkin_checkout_columns... DONE
2025_05_23_062159_add_latitude_longitude ........... DONE
```

**(Dangerous in prod) Fresh + seed attempted:**

```
php artisan migrate:fresh --seed --force
Dropping all tables ... DONE
INFO  Preparing database.
Creating migration table ... DONE
INFO  Running migrations. (same as above)
INFO  Seeding database.
Database\Seeders\HRSeeder  RUNNING

In HRSeeder.php line 14:
  Class "Faker\Factory" not found
```

**Install dev deps to unlock Faker (temporary):**

```
php composer.phar install
# Downloads fakerphp/faker and other dev-only packages...
# INFO  Discovering packages.
# ...
```

**Seed succeeded:**

```
php artisan db:seed --force
INFO  Seeding database.
Database\Seeders\HRSeeder  RUNNING
Database\Seeders\HRSeeder  49 ms DONE
```

**Return to production-only deps:**

```
php composer.phar install --no-dev --optimize-autoloader
# Removes dev packages including fakerphp/faker
# INFO  Discovering packages.
# ...
```

**Finalize caches:**

```
php artisan config:cache
php artisan route:clear
php artisan view:cache
```

---

## 4) Post-deploy checklist

* [ ] **Docroot**: Subdomain `hrm.reltroner.com` points to `.../hrm/public`
* [ ] **ENV**: `APP_URL=https://hrm.reltroner.com`, `APP_ENV=production`, `APP_DEBUG=false`
* [ ] **DB**: Credentials valid; migrations seeded
* [ ] **Permissions**:

  ```bash
  chmod -R 775 storage bootstrap/cache
  find public -type d -exec chmod 755 {} \;
  find public -type f -exec chmod 644 {} \;
  ```
* [ ] **SSL** enabled in hPanel
* [ ] **DNS** A record for `hrm` → correct server IP

Optional quick probe (remove after use):

```bash
echo "<?php header('Content-Type: text/plain'); echo 'OK';" > public/health.php
# Visit https://hrm.reltroner.com/health.php
rm -f public/health.php
```

---

## 5) Important production notes

* Avoid `migrate:fresh --seed` on production — it **drops all tables**. Prefer:

  ```bash
  php artisan migrate --force
  php artisan db:seed --force   # only if you need to insert baseline data
  ```
* Only install dev packages temporarily for seeding; revert to `--no-dev` afterward.
* If seeding requires an **admin user**, you can:

  * Keep a minimal seeder that doesn’t rely on Faker, or
  * Create the admin via `php artisan tinker` / `phpMyAdmin`.

---

## 6) Commands summary (copy-paste ready)

```bash
# From project root on the server
php artisan migrate --force

# If seeding complains about Faker:
php composer.phar install
php artisan db:seed --force
php composer.phar install --no-dev --optimize-autoloader

php artisan config:cache
php artisan route:clear
php artisan view:cache
```

---

**Author:** Rei Reltroner (Raidan Malik Sandra)
**Host:** Hostinger shared (`my-kul-web2046`) • Asia/Jakarta (UTC+7)
**Last updated:** 2025-09-01
