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
