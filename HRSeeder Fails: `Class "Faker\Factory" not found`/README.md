# 🐛 Bug Report — HRSeeder Fails: `Class "Faker\Factory" not found`

This document records the bug encountered during database seeding with the `HRSeeder` class.

---

## 1. Context

Environment:
- PHP: **8.3.23** (`php83`)
- Laravel: **v12.13.0**
- Composer: **2.8.11**
- Database: **MariaDB 10.11.13**

Command executed:

```bash
php83 artisan migrate --force --seed
````

---

## 2. Bug Encountered

**Error Output:**

```
INFO  Seeding database.

Database\Seeders\HRSeeder ......................................................................... RUNNING

In HRSeeder.php line 14:

  Class "Faker\Factory" not found
```

---

## 3. Root Cause

* The `HRSeeder.php` class imports **Faker\Factory**:

```php
use Faker\Factory as Faker;
```

* However, **Faker** is not installed by default in Laravel production dependencies.
* Laravel previously bundled `Faker` as a default dependency, but modern versions require explicit installation.

---

## 4. Steps to Reproduce

1. Run migrations and seeds with `--seed`.
2. Laravel executes `HRSeeder`.
3. The seeder attempts to use `Faker\Factory`.
4. Composer cannot resolve the class → error thrown.

---

## 5. Solution

### Install Faker

Run:

```bash
php83 $HOME/bin/composer83 require fakerphp/faker --dev
```

Or, if you want Faker available outside dev mode (e.g., seeding on production):

```bash
php83 $HOME/bin/composer83 require fakerphp/faker
```

### Verify Installation

Check that `fakerphp/faker` is now in `composer.json`:

```json
"require": {
    ...
    "fakerphp/faker": "^1.23"
}
```

---

## 6. Retry Seeder

Re-run:

```bash
php83 artisan migrate:fresh --seed
```

**Expected Result:**

* Database migrations run successfully.
* `HRSeeder` populates employees, roles, tasks, payrolls, presences, and leave requests without error.

---

## ✅ Outcome

* The bug was caused by missing `fakerphp/faker` dependency.
* Installing the Faker package resolves the error.
* Seeder now runs successfully, populating test data.
