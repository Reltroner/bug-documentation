# 🔒 Role Deletion Protection: Admin & HR Manager

This document explains the security update implemented to **prevent deletion** of critical roles such as `Admin` and `HR Manager` from both UI and backend layers.

---

## 🔍 Problem

Previously, all roles could be deleted freely via:

- Clicking the **Delete** button in the UI
- Sending a direct `DELETE` request to the endpoint

This introduced a serious risk: accidentally or maliciously deleting critical roles could break user access controls.

---

## ✅ Solution Overview

### 1. UI-Level Protection (Blade View)

In `resources/views/roles/index.blade.php`, the Delete button is conditionally hidden:

```blade
@if (!in_array($role->title, ['Admin', 'HR Manager']))
    <form action="{{ route('roles.destroy', $role->id) }}" method="POST" class="d-inline">
        @csrf
        @method('DELETE')
        <button class="btn btn-danger btn-sm">Delete</button>
    </form>
@endif
````

### 2. Backend-Level Protection (Controller Guard)

In `app/Http/Controllers/RoleController.php`, the `destroy()` method prevents deletion via direct request:

```php
public function destroy(Role $role)
{
    $protectedRoles = ['Admin', 'HR Manager'];

    if (in_array($role->title, $protectedRoles)) {
        return redirect()->route('roles.index')->with('error', "You cannot delete the '{$role->title}' role.");
    }

    $role->delete();

    return redirect()->route('roles.index')->with('success', 'Role deleted successfully.');
}
```

### 3. Error Message Handling

To display the backend error on the UI, the following block is added to the Blade view:

```blade
@if (session('error'))
<div class="alert alert-danger alert-dismissible fade show" role="alert">
    <strong>Error:</strong> {{ session('error') }}
    <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
</div>
@endif
```

---

## 🧠 Why This Matters

| Concern             | Before                | After                   |
| ------------------- | --------------------- | ----------------------- |
| Admin role deletion | ✅ Possible            | ❌ Blocked               |
| HR Manager deletion | ✅ Possible            | ❌ Blocked               |
| Manual request      | ✅ Allowed             | ❌ Rejected with message |
| UX clarity          | ❌ All roles deletable | ✅ Critical roles hidden |

---

## 🔐 Best Practice Note

Never rely solely on UI for security.

**Always enforce validation in the backend** to prevent tampering or API misuse.

---

Let Astralis light the unknown ⚡
