## 🐛 Bug: Role creation looked duplicated and later stopped inserting during debugging

**Module:** Role Management (`RoleController`, `Role` model)
**Stack:** Laravel 12, Blade, MySQL

### Summary

While testing the **Create Role** form, it looked like:

1. A single form submission was creating **two identical role records**, and
2. After adding a `dd()` call for debugging, submitting the form **no longer created any new records** in the `roles` table.

After investigation, the behavior was clarified and the controller/model were adjusted to use proper validation and mass assignment.

---

## 🔁 Steps to Reproduce (original behavior)

1. Go to **Roles → Create Role**.
2. Fill the form, for example:

   * **Role Title:** `test`
   * **Description:** `test123`
3. Click **Create Role**.
4. Return to the **Role List** page.

### Expected

* Exactly **one** new row is inserted into the `roles` table.
* The new role appears once in the Role List table.

### Actual

* It appeared as if the role was duplicated in the list.
* After adding `dd('RoleController@store dipanggil');` at the top of the `store()` method for debugging, new roles stopped being inserted entirely.

---

## 🔎 Debugging Notes

1. **Checked controller invocation with `dd()`**

   ```php
   public function store(Request $request)
   {
       dd('RoleController@store dipanggil');

       $request->validate([
           'title'       => 'required|string|max:255',
           'description' => 'nullable|string',
       ]);

       Role::create($request->all());

       return redirect()->route('roles.index')
           ->with('success', 'Role created successfully.');
   }
   ```

   * The browser showed the `dd()` output:
     `"RoleController@store dipanggil"`
   * This confirmed the **`store()` method was called exactly once** per submission.
   * However, because `dd()` means *dump and die*, the execution **stops at that line**, so `validate()` and `Role::create()` never run.
   * That explains why, during debugging, **no new rows were inserted**.

2. **Verified database state directly**

   * Opened the `roles` table in MySQL.
   * Verified that the rows matched what was shown in the UI and that no new rows were being added while `dd()` was in place.
   * Once `dd()` was removed, new roles started appearing as expected.

3. **Mass assignment configuration**

   * Ensured the `Role` model allowed the fields to be mass-assigned via `$fillable`.

   ```php
   // app/Models/Role.php
   namespace App\Models;

   use Illuminate\Database\Eloquent\Model;
   use Illuminate\Database\Eloquent\Factories\HasFactory;
   use Illuminate\Database\Eloquent\SoftDeletes;

   class Role extends Model
   {
       use HasFactory, SoftDeletes;

       protected $fillable = [
           'title',
           'description',
       ];

       /**
        * Get the employees that belong to this role.
        */
       public function employees()
       {
           return $this->hasMany(Employee::class);
       }
   }
   ```

4. **Uniqueness requirement**

   * To prevent duplicate role titles, validation was updated to use a `unique` rule on the `title` column.

---

## ✅ Final Fix

### Controller

```php
// app/Http/Controllers/RoleController.php

public function store(Request $request)
{
    $validated = $request->validate([
        'title'       => 'required|string|max:255|unique:roles,title',
        'description' => 'nullable|string',
    ]);

    Role::create($validated);

    return redirect()->route('roles.index')
        ->with('success', 'Role created successfully.');
}
```

### Model

```php
// app/Models/Role.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\SoftDeletes;

class Role extends Model
{
    use HasFactory, SoftDeletes;

    protected $fillable = [
        'title',
        'description',
    ];

    /**
     * Get the employees that belong to this role.
     */
    public function employees()
    {
        return $this->hasMany(Employee::class);
    }
}
```

---

## 📌 Lessons Learned

* `dd()` is useful for checking whether a method is called, but it **completely stops execution**, so it should only be used temporarily.
* Always verify:

  * The actual rows in the database (not only what the UI shows),
  * That the model’s `$fillable` property includes all fields used in `Model::create()`,
  * That validation rules enforce business constraints (e.g., `unique:roles,title` for role titles).
