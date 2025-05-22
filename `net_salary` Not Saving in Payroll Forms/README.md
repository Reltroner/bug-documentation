# 🧮 Bug Report: `net_salary` Not Saving in Payroll Forms

## Summary

This bug report documents an issue where the `net_salary` field in the `payrolls` module was **not being saved to the database** when using the `create` or `edit` forms, despite being included in the model's `$fillable` attributes.

---

## 🧨 Problem

- The `net_salary` value remained `null` or unchanged after submitting the form.
- No validation error was thrown.
- No input field for `net_salary` was present in the form view.

---

## 🧪 Root Cause

- **Missing Input Field**: The forms in `resources/views/payrolls/create.blade.php` and `edit.blade.php` did not include a `<input name="net_salary">` field.
- **Controller Behavior**: The controller expected all form data to be included using `$request->all()` or manual assignment, but `net_salary` was never passed in the request.
- **Optional Logic Conflict**: In some systems, `net_salary` is calculated automatically from `salary + bonus - deduction`, but this was not handled either in controller, model accessor, or frontend JS.

---

## ✅ Solution

### 1. Add a Form Field (optional input)

```blade
<div class="mb-3">
    <label for="net_salary" class="form-label">Net Salary</label>
    <input type="number" step="0.01" class="form-control @error('net_salary') is-invalid @enderror"
           id="net_salary" name="net_salary"
           value="{{ old('net_salary', $payroll->net_salary ?? '') }}">
    @error('net_salary')
        <div class="invalid-feedback">{{ $message }}</div>
    @enderror
</div>
````

### 2. Automatically Calculate in JavaScript (recommended)

```blade
<input type="number" step="0.01" id="net_salary" name="net_salary" class="form-control" readonly>

<script>
    function calculateNetSalary() {
        const salary = parseFloat(document.getElementById('salary').value) || 0;
        const bonus = parseFloat(document.getElementById('bonus').value) || 0;
        const deduction = parseFloat(document.getElementById('deduction').value) || 0;
        document.getElementById('net_salary').value = (salary + bonus - deduction).toFixed(2);
    }

    document.getElementById('salary').addEventListener('input', calculateNetSalary);
    document.getElementById('bonus').addEventListener('input', calculateNetSalary);
    document.getElementById('deduction').addEventListener('input', calculateNetSalary);
</script>
```

### 3. Controller Must Receive and Save It

```php
Payroll::create([
    'employee_id' => $request->employee_id,
    'salary' => $request->salary,
    'bonus' => $request->bonus,
    'deduction' => $request->deduction,
    'net_salary' => $request->net_salary, // Make sure this is passed!
    'payment_date' => $request->payment_date,
]);
```

---

## 🚫 Optional Alternative (Model Accessor Only)

You may choose to remove `net_salary` from the DB entirely and use an accessor instead:

```php
public function getNetSalaryAttribute()
{
    return ($this->salary ?? 0) + ($this->bonus ?? 0) - ($this->deduction ?? 0);
}
```

This way, you only store raw values and let Laravel compute `net_salary` dynamically. Do not include it in the form if you take this route.

---

## 🧠 Lessons Learned

* Always match model `$fillable` fields with form inputs.
* Computed values like `net_salary` should either be:

  * Explicitly passed from the frontend, or
  * Computed and stored in the backend, or
  * Handled via model accessors (read-only).

---

## Status

✅ Fixed and verified
💡 `net_salary` is now either editable, auto-calculated, or dynamically computed depending on design choice.

---

## Related Files

* `resources/views/payrolls/create.blade.php`
* `resources/views/payrolls/edit.blade.php`
* `app/Http/Controllers/PayrollController.php`
* `app/Models/Payroll.php`

