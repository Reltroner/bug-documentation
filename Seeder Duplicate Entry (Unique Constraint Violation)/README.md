# 🐛 Bug Report — Seeder Duplicate Entry (Unique Constraint Violation)

This document records the issue where running database seeders caused a **duplicate primary key error** in Laravel.

---

## 1. Problem Description

Environment:
- Domain: `https://hrm.reltroner.com`
- PHP: **8.3.24**
- Laravel: **v12.13.0**
- Command executed:

```bash
php83 artisan migrate --force --seed
````

**Error Output:**

```
Illuminate\Database\UniqueConstraintViolationException

SQLSTATE[23000]: Integrity constraint violation: 1062 Duplicate entry '1' for key 'PRIMARY'
...
insert into `departments` (...)
```

---

## 2. Root Cause

* The `HRSeeder` attempted to insert rows with fixed primary keys (`id = 1, 2, 3`) into the `departments` table.
* Since the seeder had already run once, running it again caused a **duplicate entry violation**.
* Seeder code used `DB::table()->insert([...])` which is **not idempotent**.

---

## 3. Solutions

### Option A — Reset Database (Fastest, destroys existing data)

If you are safe to reset all data:

```bash
cd ~/repositories/reltroner-hr-app
php83 artisan migrate:fresh --seed --force
```

This drops all tables, recreates them, and reseeds with fresh data.

---

### Option B — Make Seeder Idempotent (Safe for re-runs)

Modify `HRSeeder.php` to use **`upsert`** instead of `insert`:

```php
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Carbon;

public function run(): void
{
    $now = Carbon::now();

    DB::table('departments')->upsert([
        ['id' => 1, 'name' => 'Engineering', 'description' => 'Handles all technical tasks.',  'status' => 'active', 'created_at' => $now, 'updated_at' => $now],
        ['id' => 2, 'name' => 'HR',          'description' => 'Manages employee welfare.',      'status' => 'active', 'created_at' => $now, 'updated_at' => $now],
        ['id' => 3, 'name' => 'Finance',     'description' => 'Handles all financial matters.', 'status' => 'active', 'created_at' => $now, 'updated_at' => $now],
    ], ['id'], ['name','description','status','updated_at']);
}
```

Then re-run:

```bash
php83 artisan db:seed --class="Database\\Seeders\\HRSeeder" --force
```

---

### Option C — Truncate Conflicting Tables Only

If you want to keep other data but reseed one table:

```bash
mysql -h 127.0.0.1 -u ynoqpang_admin -p'<your-password>' \
  -e "TRUNCATE TABLE ynoqpang_reltronerhrapp.departments;"

php83 artisan db:seed --class="Database\\Seeders\\HRSeeder" --force
```

Repeat for any other conflicting tables (roles, employees, etc.) as needed.

---

## 4. Optional Cleanup

* Remove temporary version check files:

```bash
rm ~/repositories/reltroner-hr-app/public/_vcheck.php
```

* Cache Laravel configuration and routes:

```bash
php83 artisan config:cache
php83 artisan route:cache
php83 artisan view:cache
```

---

## ✅ Resolution Status

* **Cause:** Seeder used non-idempotent inserts with fixed IDs.
* **Fix Options:**

  * Reset database (`migrate:fresh`).
  * Update seeder to use `upsert`.
  * Truncate conflicting tables before reseeding.
* **Outcome:** Seeder runs without duplicate key errors.

