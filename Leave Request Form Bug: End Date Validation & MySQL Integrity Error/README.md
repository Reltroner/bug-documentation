# Leave Request Form Bug: End Date Validation & MySQL Integrity Error

## 🐞 Bug Description

When submitting the Leave Request form, if the **End Date** field is left empty, the application throws the following error:

```

SQLSTATE\[23000]: Integrity constraint violation: 1048
Column 'end\_date' cannot be null
(Connection: mysql, SQL: insert into `leave_requests` ...)

````

**Symptoms:**
- The input field for End Date is highlighted in red with an error message ("The end date field is required.") in the UI.
- Despite this, a backend error occurs, and the request fails with a SQL exception.

## 💡 Root Cause

- The **end_date** column in the `leave_requests` table is set to `NOT NULL`.
- There was **no validation rule** for the `end_date` field in the Laravel controller, allowing a null value to be sent to the database.
- MySQL rejects null for a `NOT NULL` column, causing an Integrity Constraint Violation.

## ✅ Solution

1. **Add validation rules in the controller:**
   - Ensure the `end_date` is required and is a valid date, and (optionally) comes after or equals the `start_date`.

   ```php
   $request->validate([
       'leave_type'  => 'required|string|max:255',
       'start_date'  => 'required|date',
       'end_date'    => 'required|date|after_or_equal:start_date',
       // ... other fields
   ]);
   ````

2. **UI Feedback:**

   * The Blade template already shows proper error feedback using `@error('end_date') ... @enderror`. With correct validation, users will see the error message and field highlight *before* the form is submitted to the database.

3. **Database Consistency:**

   * With validation in place, only requests with a valid, non-null `end_date` are submitted, preventing integrity errors.

## 📋 Example Fix

**Before (missing validation):**

   ```php
   // No validation for 'end_date'
   ```

**After (fixed):**

   ```php
   $request->validate([
       // ...
       'end_date' => 'required|date|after_or_equal:start_date',
   ]);
   ```

## 🚦 Result

* Submitting the form without an End Date now triggers an **inline validation error**, blocking submission until the field is filled.
* No more SQL integrity errors caused by null `end_date` values.

---

> For further details, see `/app/Http/Controllers/LeaveRequestController.php` and `/resources/views/leave_requests/create.blade.php`.
