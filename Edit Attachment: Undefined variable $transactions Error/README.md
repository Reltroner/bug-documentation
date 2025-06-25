## 🐛 Edit Attachment: `Undefined variable $transactions` Error

### Overview

When accessing the **edit page** for an attachment, you may receive the following error:

```
ErrorException
Undefined variable $transactions
```

This usually happens at the Blade line where the edit form tries to loop over `$transactions` to build the “Related Transaction” dropdown:

```blade
@foreach ($transactions as $transaction)
    <!-- ... -->
@endforeach
```

### Root Cause

The `$transactions` variable is **not defined or passed** to the view.
This typically means the **controller method** for the edit route is missing the following:

* A database query for all transactions.
* A `compact('transactions')` when returning the view.

### Solution

#### 1. Update Your Controller Method

Open `app/Http/Controllers/AttachmentController.php`
Find the `edit` method, and update it to pass the `$transactions` variable to the view:

```php
public function edit(Attachment $attachment)
{
    // Add this line:
    $transactions = \App\Models\Transaction::all();

    return view('attachments.edit', compact('attachment', 'transactions'));
}
```

#### 2. Why This Works

Now, your Blade file will always receive a `$transactions` variable, avoiding the error and properly populating the dropdown menu.

### Prevention

* Always check which variables your Blade views require.
* Use strong controller-view contracts: every variable referenced in the Blade file should be passed explicitly from the controller.

### Example (before/after)

**Before:**

```php
public function edit(Attachment $attachment)
{
    return view('attachments.edit', compact('attachment'));
}
```

**After (Fixed):**

```php
public function edit(Attachment $attachment)
{
    $transactions = \App\Models\Transaction::all();
    return view('attachments.edit', compact('attachment', 'transactions'));
}
```

### References

* [Laravel Controllers - Passing Data to Views](https://laravel.com/docs/views#passing-data-to-views)

---

**Status:**
Bug resolved by ensuring all required variables are passed to the Blade template from the controller.
