# 🐞 Known Bug: `$leave_requests` Undefined Variable

### 🧩 Description

When accessing the `leave_requests.index` view, the following error appears:

```
Undefined variable $leave_requests
```

### 📍 Root Cause

In the `LeaveRequestController@index` method, the variable passed to the view was incorrectly named:

```php
// Before (Incorrect)
return view('leave_requests.index', compact('leaveRequests'));

// View expects: $leave_requests (snake_case)
```

This causes Laravel to throw an undefined variable error because the Blade view is using `$leave_requests`, but the controller only passed `$leaveRequests`.

### ✅ Fix

Update the controller to use the correct variable name and match the Blade view:

```php
// After (Correct)
$leave_requests = LeaveRequest::with('employee')->latest()->get();
return view('leave_requests.index', compact('leave_requests'));
```

Alternatively, you can rename the variable in the Blade file to match what was passed from the controller—but using snake\_case is the preferred Laravel convention for view variables.

### 📌 Affected File

* `resources/views/leave_requests/index.blade.php`
* `app/Http/Controllers/LeaveRequestController.php`

---

## 🐞 Bug Report: Undefined Variable `$leaveRequest` on Edit Leave Request Page

### 🔍 Description

An `Undefined variable $leaveRequest` error is triggered when accessing the **Edit Leave Request** form via the `leave_requests.index` view.

**Error location:**
`resources/views/leave_requests/edit.blade.php`

**Error message:**

```
Undefined variable $leaveRequest
```

### 📍 Root Cause

The controller method `LeaveRequestController::edit()` passes the variable using snake\_case:

```php
return view('leave_requests.edit', compact('leave_request'));
```

However, in the Blade template, the variable is referenced in camelCase:

```blade
<!-- This will throw undefined variable error -->
{{ $leaveRequest->id }}
```

### ✅ Resolution

You have two options:

#### Option A: Update the controller

Use camelCase to match Blade usage:

```php
return view('leave_requests.edit', ['leaveRequest' => $leave_request, 'employees' => $employees]);
```

#### Option B: Update the Blade view

Use the variable as it is (`$leave_request`) throughout:

```blade
<form action="{{ route('leave_requests.update', $leave_request->id) }}" method="POST">
...
value="{{ old('start_date', $leave_request->start_date->format('Y-m-d')) }}"
```

### 🛠 Recommended Fix

For consistency with Laravel model route-binding conventions, prefer **Option B** and stick with snake\_case (`$leave_request`) when using implicit model binding.

---
