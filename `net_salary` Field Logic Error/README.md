# 🧮 Bug Report: `net_salary` Field Logic Error

## Summary

A critical logic issue was found in the payroll form submission system. The `net_salary` field was editable by users and submitted directly to the database, allowing incorrect or illogical values (e.g. entering `6` instead of auto-calculated net salary). This compromised the integrity of payroll records.

---

## ❌ Problem

### Symptoms
- `net_salary` field is editable on the **create** and **edit** forms.
- Users could submit invalid values, such as `6`, regardless of the values in `salary`, `bonus`, or `deduction`.
- The database accepted and stored this invalid value because it passed validation rules.
- It caused discrepancies between expected calculations and actual stored values.

### Example:
| Salary | Bonus | Deduction | Net Salary (Actual) | Net Salary (Wrong) |
|--------|-------|-----------|----------------------|---------------------|
| 9224.40 | 270.24 | 526.77 | ✅ 8,967.87 | ❌ 6 |

---

## ✅ Recommended Fix

### Option A — 🔐 **Fix in Controller** (Recommended for DB persistence)

Move the `net_salary` calculation to the controller so that it's **always derived**, never manually input.

### Updated Controller Logic

In both `store()` and `update()` methods of `PayrollController.php`, use:

```php
$data['bonus'] = $data['bonus'] ?? 0;
$data['deduction'] = $data['deduction'] ?? 0;
$data['net_salary'] = $data['salary'] + $data['bonus'] - $data['deduction'];
````

### Remove Manual Entry from Form (Optional but Recommended)

* Set `net_salary` field as `readonly` or remove it completely from the form to prevent manual override.
* Optionally show the value in a `<span>` or hidden input.

---

## ✅ Results After Fix

* No invalid `net_salary` value can be stored.
* Ensures math consistency across all payroll records.
* Improved UX by calculating and displaying the value in real time if needed.

---

## Related Files

* `app/Http/Controllers/PayrollController.php`
* `resources/views/payrolls/create.blade.php`
* `resources/views/payrolls/edit.blade.php`
* `app/Models/Payroll.php`

---

## Status

✅ Fixed and deployed
🧪 Fully tested via manual form submissions
🔒 Data integrity now enforced at controller level

---

> Let Astralis light the unknown.

