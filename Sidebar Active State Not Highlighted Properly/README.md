# 🐛 Sidebar Active State Not Highlighted Properly

## Description

In this project using Laravel + Blade + Mazer UI, sidebar items like **Tasks**, **Employees**, etc., were not being highlighted when visiting their respective pages.

This was due to the sidebar links using static classes like:

```blade
<li class="sidebar-item">
    <a href="index.html" class="sidebar-link">
        <i class="bi bi-list-task"></i>
        <span>Tasks</span>
    </a>
</li>
````

No logic was applied to assign the `active` class based on the current route.

---

## Expected Behavior

When visiting `/tasks`, the corresponding sidebar item should be highlighted with the `active` class:

```html
<li class="sidebar-item active">...</li>
```

---

## Solution ✅

Update the sidebar items to use `request()->is()` or `Route::is()` for dynamic class assignment:

```blade
<li class="sidebar-item {{ request()->is('tasks*') ? 'active' : '' }}">
    <a href="{{ route('tasks.index') }}" class="sidebar-link">
        <i class="bi bi-list-task"></i>
        <span>Tasks</span>
    </a>
</li>
```

This ensures the class `active` is only applied when the current route matches a pattern like `tasks`, `tasks/create`, `tasks/123/edit`, etc.

---

## Related Routes

Ensure named routes exist for each section:

```php
Route::resource('tasks', TaskController::class);
Route::resource('employees', EmployeeController::class);
// ...other resources
```

---

## Outcome

After applying this fix, the sidebar UI responds correctly to navigation context and highlights the current page—enhancing user experience and visual clarity.
