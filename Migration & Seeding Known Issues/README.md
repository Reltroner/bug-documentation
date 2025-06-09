# 🐛 Migration & Seeding Known Issues

### 1. **Foreign Key Constraint Error**

**Error Message:**

```
SQLSTATE[HY000]: General error: 1824 Failed to open the referenced table 'cost_centers' ...
```

**Root Cause:**

* The migration for `transaction_details` references the `cost_centers` table, but `cost_centers` was not created yet (incorrect migration order).

**How to Fix:**

* **Make sure the migration file for `cost_centers` runs BEFORE `transaction_details`.**
* Rename migration files or adjust timestamps so that the `cost_centers` table is created earlier.
* Example migration file order:

  ```
  2025_06_09_142400_create_cost_centers_table.php
  2025_06_09_142435_create_transaction_details_table.php
  ```
* Run:

  ```bash
  php artisan migrate:fresh --seed
  ```
* **Reason:** MySQL (and Laravel) requires the referenced table in a foreign key to exist before the constraint is created.

---

### 2. **Missing Timestamp Columns (`created_at`, `updated_at`)**

**Error Message:**

```
SQLSTATE[42S22]: Column not found: 1054 Unknown column 'updated_at' in 'field list'
```

**Root Cause:**

* The migration for `attachments` (or any other table) is missing `$table->timestamps();`.
* Laravel’s Eloquent (and factory/seeder) always tries to insert `created_at` and `updated_at` by default.

**How to Fix:**

* In the migration for each affected table, add:

  ```php
  $table->timestamps();
  ```
* For example, your `attachments` table should look like:

  ```php
  Schema::create('attachments', function (Blueprint $table) {
      $table->id();
      $table->foreignId('transaction_id')->constrained('transactions')->onDelete('cascade');
      $table->string('file_path');
      $table->string('file_name');
      $table->timestamp('uploaded_at')->nullable();
      $table->timestamps(); // <-- Add this!
  });
  ```
* Run:

  ```bash
  php artisan migrate:fresh --seed
  ```
* **Reason:** All Eloquent-based seeders/factories require those timestamp columns unless you explicitly disable them.

---

### 3. **General Best Practices**

* Always check migration file order if using foreign keys.
* Always add `$table->timestamps();` for tables that will be seeded or use Eloquent.
* If you update any migration file, always use:

  ```bash
  php artisan migrate:fresh --seed
  ```

  to fully reset and repopulate your schema.

---

### 4. **How to Check Migration Order**

You can check migration order using:

```bash
php artisan migrate:status
```

Or manually verify filenames in `database/migrations/` are prefixed to guarantee creation order.

---

### 5. **If Other Errors Occur**

* Read the SQL error carefully for missing columns or missing referenced tables.
* Most issues in `php artisan migrate:fresh --seed` are due to migration order or missing fields.
* Fix the order or add required columns, then run migrate\:fresh again.

---

**If you keep these best practices, your migrations and seeders will be reliable and maintainable.**
Feel free to report new issues or PR fixes if you encounter other bugs!

---

Let Astralis light the unknown 🚀
