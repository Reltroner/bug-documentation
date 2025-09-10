# Laravel Migration Bug: Foreign Key Order Issue

This repository documents a migration bug encountered while integrating **Laravel Sanctum** and creating related tables (`attendances`, `shifts`, and `attendance_details`) in a fresh Laravel project.

---

## ⚠️ Problem

When running:

```bash
php artisan migrate
````

The following error occurred:

```
SQLSTATE[HY000]: General error: 1824 Failed to open the referenced table 'shifts'
(Connection: mysql, SQL: alter table `attendances` add constraint 
`attendances_shift_id_foreign` foreign key (`shift_id`) references `shifts` (`id`) on delete set null)
```

This happened because the migration for **`attendances`** was executed **before** the migration for **`shifts`**, even though `attendances` has a foreign key to `shifts`.

---

## 🔍 Root Cause

* `attendances` references `shifts` with a nullable FK (`shift_id`).
* MySQL requires the referenced table (`shifts`) to exist first.
* Laravel determines migration order by **filename timestamp**, so if the `attendances` file has an earlier timestamp, it runs before `shifts`.

---

## ✅ Solutions

### Option 1 – Quick Fix (One-Time)

Run the `shifts` migration manually first:

```bash
# Step 1: Create `shifts` table first
php artisan migrate --path=database/migrations/2025_09_10_045450_create_shifts_table.php

# Step 2: Continue the remaining migrations (including `attendances`)
php artisan migrate
```

---

### Option 2 – Proper Fix (Recommended)

Rename migration files so `shifts` is created **before** `attendances`:

1. Rename `shifts` migration file to an earlier timestamp than `attendances`.
   Example:

   ```
   2025_09_10_042200_create_shifts_table.php
   2025_09_10_042234_create_attendances_table.php
   ```

2. Reset and rerun migrations:

```bash
php artisan migrate:fresh
```

Now, Laravel runs them in the correct order.

---

## 📂 Bonus: Related Migrations

### `attendance_details` (One-to-Many with `attendances`)

```php
Schema::create('attendance_details', function (Blueprint $table) {
    $table->id();
    $table->foreignId('attendance_id')->constrained()->cascadeOnDelete();
    $table->enum('type', ['check_in','check_out','break_start','break_end'])->index();
    $table->timestamp('occurred_at'); // UTC
    $table->string('source', 20)->default('mobile'); // mobile|web|admin
    $table->decimal('lat', 10, 7)->nullable();
    $table->decimal('lng', 10, 7)->nullable();
    $table->string('device_id', 100)->nullable();
    $table->timestamps();
});
```

---

### Seeder Example for `shifts`

```php
DB::table('shifts')->insert([
    [
        'name' => 'Office 9–5',
        'start_at' => '09:00:00',
        'end_at' => '17:00:00',
        'grace_minutes' => 15,
        'work_minutes_target' => 480,
        'created_at' => now(),
        'updated_at' => now(),
    ],
    [
        'name' => 'Night 10–6',
        'start_at' => '22:00:00',
        'end_at' => '06:00:00', // crosses midnight
        'grace_minutes' => 15,
        'work_minutes_target' => 480,
        'created_at' => now(),
        'updated_at' => now(),
    ],
]);
```

Run seeder:

```bash
php artisan db:seed --class=ShiftSeeder
```

---

## 📝 Summary

* Error `1824 Failed to open the referenced table 'shifts'` occurs due to **migration order**.
* **Quick Fix**: Run `shifts` migration first via `--path`.
* **Recommended Fix**: Rename migration timestamps and rerun with `php artisan migrate:fresh`.

---

### References

* [Laravel Migrations Documentation](https://laravel.com/docs/migrations)
* [MySQL Foreign Key Constraints](https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html)

