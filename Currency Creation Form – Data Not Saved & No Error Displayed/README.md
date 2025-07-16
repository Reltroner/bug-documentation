## Currency Creation Form – Data Not Saved & No Error Displayed

### Overview

When submitting the **Create Currency** form (`/currencies/create`), the page reloads but:

* No data is saved to the database.
* The user stays on the same page.
* No error message or feedback is displayed.

This makes it difficult for users to know what went wrong or how to proceed.

---

### Steps to Reproduce

1. Go to `http://127.0.0.1:8000/currencies/create`
2. Fill out all fields in the **New Currency Form** (e.g.):

   * Currency Code: `$DEL`
   * Currency Name: `Depcutland's Currency`
   * Symbol (Optional): `depcut`
   * Active: checked
3. Click **Save Currency**

---

### Expected Behavior

* After clicking **Save Currency**, the new currency should be saved to the database.
* The user should be redirected to the currencies list, or see a success notification.
* If there is any error (validation or server), an appropriate error message should be displayed on the form.

---

### Actual Behavior

* The form page reloads.
* No data is added to the database.
* No error message is displayed (neither success nor failure).
* User receives no feedback.

---

### Technical Details

* **Route:** `POST /currencies`
* **Blade file:** `resources/views/currencies/create.blade.php`
* **Controller:** `CurrencyController@store`
* **Framework:** Laravel 12.17.0
* **Observed with:** PHP 8.4.4, Localhost (Laragon), tested in latest Chromium browser

---

### Investigation & Possible Causes

* **Missing required field:**
  The controller requires a `rate` field (`'rate' => 'required|numeric|min:0',`) but the form does **not** include an input for `rate`.
  This triggers validation failure, so the controller never creates the record, but the error is not shown because:

  * There is no global error display in the Blade template (only per-field errors).
  * The user does not know which field is missing, since it’s not visible on the form.

* **No global error display:**
  Only per-field errors are displayed, so validation errors not tied to visible fields are never shown to the user.

---

### How to Fix

1. **Add the missing `rate` field to the form**
   Add a number input for `rate` (required, numeric, min 0).

2. **Add a global error alert block**
   Show all validation errors at the top of the form.

3. **Optional:**

   * Autofocus on the first field.
   * Use `old('rate', 1)` as a default value for the rate if applicable.

---

### Example of Fixed Form Snippet

```blade
{{-- Add this field inside your form --}}
<div class="mb-3">
    <label for="rate" class="form-label">Currency Rate <span class="text-danger">*</span></label>
    <input type="number" name="rate" id="rate" step="0.0001" min="0"
        class="form-control @error('rate') is-invalid @enderror"
        value="{{ old('rate', 1) }}" required>
    @error('rate')
        <div class="invalid-feedback">{{ $message }}</div>
    @enderror
</div>
```

---

### Related Files

* `resources/views/currencies/create.blade.php`
* `app/Http/Controllers/CurrencyController.php`

---

### Status

* [ ] **Open**
* [ ] **Needs Validation Field**
* [ ] **Needs Error Display**
* [ ] **Ready for Retest**

---

**If you are facing this bug, please update your Blade file as above and ensure your form matches the controller’s validation rules.**

---

Let Astralis light the unknown.
— Reltroner Studio Engineering
