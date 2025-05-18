# 🐛 Bug: Date Fields Empty in Edit Form

## Problem

In the edit form for Employees (`resources/views/employees/edit.blade.php`), the following fields:

- `Birth Date`
- `Hire Date`

appear empty even though the values exist in the database and are passed correctly to the view.

---

## Cause

The `<input type="date">` element **requires a value in `YYYY-MM-DD` format**. If the value passed (e.g. a `Carbon` instance or string) is not in this format, the field will appear blank.

Laravel's `{{ $employee->birth_date }}` may return a `Carbon` object or localized string, which is not valid for HTML5 `<input type="date">`.

---

## Fix ✅

Use the `.format('Y-m-d')` method on the date fields to ensure compatibility:

```blade
<input type="date"
       name="birth_date"
       id="birth_date"
       value="{{ old('birth_date', $employee->birth_date ? $employee->birth_date->format('Y-m-d') : '') }}"
       class="form-control" />

<input type="date"
       name="hire_date"
       id="hire_date"
       value="{{ old('hire_date', $employee->hire_date ? $employee->hire_date->format('Y-m-d') : '') }}"
       class="form-control" />
````

> Note: Make sure `birth_date` and `hire_date` are casted to `date` in the model:

```php
// app/Models/Employee.php
protected $casts = [
    'birth_date' => 'date',
    'hire_date' => 'date',
];
```

---

## Result

Now when accessing the edit form, the date fields show the correct values and are compatible with HTML5 date inputs.

---

## Related Files

* `resources/views/employees/edit.blade.php`
* `app/Models/Employee.php`
