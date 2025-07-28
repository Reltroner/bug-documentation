# \:bug: Payments Table — Action Buttons Not Aligned (View/Edit/Delete)

### Overview

In the Payments module, the **View**, **Edit**, and **Delete** action buttons in the table do not appear horizontally aligned or visually grouped as shown in the Invoices module (see reference screenshot).
This leads to inconsistent user experience and a less professional appearance, especially when compared to other modules.

---

### Steps to Reproduce

1. Go to `/payments` in the app.
2. Observe the **Action** column in the Payment List table.
3. The **View**, **Edit**, and **Delete** buttons are not aligned as a horizontal group (may appear stacked or with inconsistent spacing).

---

### Expected Behavior

* All action buttons (**View**, **Edit**, **Delete**) are shown in a single row, equally spaced, just like the Invoices table.
* The button layout is consistent, compact, and visually pleasing on all screen sizes.

---

### Actual Behavior

* Buttons appear misaligned, stacked, or not evenly spaced.
* This inconsistency impacts the professional look and usability of the UI.

---

### Root Cause

* Each button is rendered separately (with forms for Delete) without being wrapped in a flex/group container.
* No flexbox or horizontal grouping is used for layout.

---

### How to Fix

Wrap all three action buttons inside a **flexbox container** (`<div class="d-flex gap-1">...</div>`) to ensure they appear in one row, with even spacing.

**Corrected Blade snippet:**

```blade
<td>
    <div class="d-flex gap-1">
        <a href="{{ route('payments.show', $payment->id) }}" class="btn btn-info btn-sm">View</a>
        <a href="{{ route('payments.edit', $payment->id) }}" class="btn btn-warning btn-sm">Edit</a>
        <form action="{{ route('payments.destroy', $payment->id) }}" method="POST" class="d-inline"
              onsubmit="return confirm('Are you sure you want to delete this payment?');" style="display:inline;">
            @csrf
            @method('DELETE')
            <button class="btn btn-danger btn-sm">Delete</button>
        </form>
    </div>
</td>
```

* This ensures all buttons align horizontally with consistent gaps, following the Bootstrap 5 pattern.

---

### Related Files

* `resources/views/payments/index.blade.php`

---

### Status

* [ ] **Open**
* [x] **Patch Ready**
* [ ] **Needs Review**
* [ ] **Closed**

---

**Reference:**
See Invoices module for best practice button grouping (`/invoices`).

---

Let Astralis light the unknown.
— Reltroner Studio Engineering
