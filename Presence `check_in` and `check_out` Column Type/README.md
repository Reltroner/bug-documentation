# 🛠️ Fix: Presence `check_in` and `check_out` Displaying as `00:00`

## Problem

This update addresses an issue in the `presences` table where the `check_in` and `check_out` fields were initially created with the `DATE` type instead of the more appropriate `DATETIME` type. As a result, only the date (`Y-m-d`) was stored in the database, and the time (`H:i`) component was lost, causing check-in and check-out times to appear as `00:00`.
The main reason why **`check_in`** and **`check_out`** fields in the `presences` table were always showing `00:00` despite values being saved in the database was because both columns were of type **`DATE`** instead of **`DATETIME`**.

---

## ✅ Solution

To ensure `check_in` and `check_out` times are stored and displayed correctly, follow these steps:

---

### 🔧 1. Update `presences` Table Column Types

Convert the `check_in` and `check_out` columns from `date` to `datetime`.

Generate a migration:

```bash
php artisan make:migration update_checkin_checkout_columns_to_datetime_in_presences_table
````

Migration file content:

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void {
        Schema::table('presences', function (Blueprint $table) {
            $table->dateTime('check_in')->change();
            $table->dateTime('check_out')->change();
        });
    }

    public function down(): void {
        Schema::table('presences', function (Blueprint $table) {
            $table->date('check_in')->change();
            $table->date('check_out')->change();
        });
    }
};
```

Run the migration:

```bash
php artisan migrate
```

---

### 🔍 2. Review and Fix the Seeder

Ensure the seeder generates values with both date and time:

```php
// Seed Presences
    for ($i = 0; $i < 30; $i++) {
        // Generate check-in between 08:00 to 10:00, 30 days ago until today
        $checkIn = $faker->dateTimeBetween('-30 days 08:00:00', 'now 10:00:00');
        $checkOut = (clone $checkIn)->modify('+8 hours');
        $dateOnly = $checkIn->format('Y-m-d');
    
        DB::table('presences')->insert([
            'employee_id' => rand(1, 10),
            'check_in'    => $checkIn->format('Y-m-d H:i:s'),
            'check_out'   => $checkOut->format('Y-m-d H:i:s'),
            'date'        => $dateOnly,
            'status'      => $faker->randomElement(['present', 'absent', 'late', 'leave']),
            'created_at'  => now(),
            'updated_at'  => now(),
        ]);
    }
```

---

### 🧠 3. Confirm Flatpickr Input Format

Ensure Flatpickr is initialized properly with date and time enabled:

```js
flatpickr(".datetime", {
    enableTime: true,
    dateFormat: "Y-m-d H:i",
    time_24hr: true,
});
```

---

### 🧪 4. Verify Model Casting in `Presence.php`

Ensure the model casts datetime fields correctly:

```php
protected $casts = [
    'check_in'  => 'datetime:Y-m-d H:i',
    'check_out' => 'datetime:Y-m-d H:i',
    'date'      => 'date:Y-m-d',
];
```

---

## 🔚 Final Result

Once all of the above is implemented:

* `check_in` and `check_out` will no longer display `00:00`.
* Timestamps will be stored and shown with minute precision.
* Flatpickr will provide accurate datetime input from users.

After making all these changes, feel free to reseed your data or test manual form input. If needed, you can always generate new migration or seeder files to cleanly reset the data.

