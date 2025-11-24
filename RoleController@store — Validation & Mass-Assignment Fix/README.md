# 🛠️ Debug Log — Reltroner HRM System

*Documented Issues & Fixes (Excluding Double-Submit Form Bug)*

This document records resolved technical issues encountered during development of **Reltroner HRM**, including database schema conflicts, model assignment issues, validation improvements, and seeding logic corrections.

It serves as a reference for future maintenance, scalability decisions, and onboarding new contributors.

---

## 1. `RoleController@store` — Validation & Mass-Assignment Fix

### **Issue**

The original controller used:

```php
Role::create($request->all());
```

This allowed unintended field submission and did not prevent duplicate entries.

### **Root Cause**

* Lack of explicit whitelisting (`$fillable` OK, but controller still too permissive)
* No uniqueness validation

### **Fix**

```php
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

---

## 2. Incorrect Query Ordering in Payroll Creation View

### **Error**

```
SQLSTATE[42S22]: Column not found: Unknown column 'name' in 'order clause'
```

### **Cause**

Employees were being sorted using a non-existent field:

```php
Employee::orderBy('name')->get();
```

However, the actual model schema uses `fullname`.

### **Fix**

```php
$employees = Employee::orderBy('fullname')->get();
```

Applied in both:

* `PayrollController@create()`
* `PayrollController@edit()`

---

## 3. Migration Duplicate Column Definition

### **Issue**

Inside the `departments` migration, the `name` column was accidentally defined twice:

```php
$table->string('name');
...
$table->string('name')->unique();
```

### **Consequence**

* Schema redundancy
* Potential unexpected migration overwrites

### **Fix**

Updated migration:

```php
$table->string('name')->unique();
```

---

## 4. Seeder Failure: Unique Constraint Violation in `presences` Table

### **Error**

```
Integrity constraint violation: Duplicate entry `employee_id + date`
```

### **Cause**

Seeder generated random attendance records without checking uniqueness, conflicting with:

```php
$table->unique(['employee_id', 'date']);
```

### **Fix**

Presence seeding logic refactored using `firstOrCreate()`:

```php
Presence::firstOrCreate(
    ['employee_id' => $employeeId, 'date' => $date],
    [
        'check_in'   => $checkIn,
        'check_out'  => $checkOut,
        'status'     => $faker->randomElement(['present', 'absent', 'late', 'leave']),
        'latitude'   => $latitude,
        'longitude'  => $longitude,
        'created_at' => now(),
        'updated_at' => now(),
    ]
);
```

### **Outcome**

* Seeder now generates realistic but non-conflicting data
* Compatible with business rule: **one attendance per employee per date**

---

## 5. Best Practice Improvements Implemented

| Area                  | Before                   | After                            |
| --------------------- | ------------------------ | -------------------------------- |
| Controller validation | Partial                  | Strict + unique + typed          |
| Model assignments     | Mix of `$request->all()` | `validated` / `only([...])`      |
| Seeder stability      | Random collisions        | Collision-safe generation        |
| DB integrity          | Weak protections         | Unique constraints enforced      |
| UX consistency        | Multiple display issues  | Normalized ordering (`fullname`) |

---

## 📌 Summary

All documented bugs were resolved with the following themes:

* **Strong validation**
* **Strict mass-assignment rules**
* **Database integrity enforcement**
* **Cleaner seed data logic**
* **Schema consistency**

These changes improve security, prevent silent data corruption, and align the system with scalable enterprise HR software standards.
