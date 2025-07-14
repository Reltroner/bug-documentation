## 🐛 Cost Center Index Page: Undefined Variable `$costcenters`

### Overview

When visiting the Cost Centers index page, you may encounter the following error:

```
ErrorException
Undefined variable $costcenters
```

This error appears at any line where the Blade template tries to use `$costcenters`, such as:

```blade
@forelse ($costcenters as $costcenter)
    ...
@endforelse
```

or

```blade
{{ $costcenters->links() }}
```

---

### Root Cause

The controller’s `index()` method sends data to the view using a variable named `$costCenters` (with a capital “C”), but the Blade template is expecting `$costcenters` (all lowercase).
**PHP variable names are case-sensitive, so `$costCenters` is not the same as `$costcenters`.**
This mismatch leads to the variable being undefined in the Blade view.

---

### Solution

**Use consistent variable names (including letter case) in both the controller and the Blade view.**

#### Example Fix

* **Before (Controller):**

  ```php
  return view('costcenters.index', compact('costCenters'));
  ```

  **Blade:**

  ```blade
  @forelse ($costcenters as $costcenter)
  ```
* **After (Controller):**

  ```php
  $costcenters = $query->orderBy('name')->paginate(20);
  return view('costcenters.index', compact('costcenters'));
  ```

  **Blade (unchanged):**

  ```blade
  @forelse ($costcenters as $costcenter)
  ```

**Alternatively:**
You could change the Blade template to use `$costCenters` everywhere, but best practice is to stick with all lowercase and keep it consistent for readability.

---

### Prevention

* **Always use the exact same variable name in both your controller and Blade views.**
* Adopt a naming convention (e.g. all lowercase or camelCase) for variables passed to views across your codebase.
* Review controller and Blade files together when refactoring resource collections.

---

### References

* [Laravel Views: Passing Data to Views](https://laravel.com/docs/views#passing-data-to-views)
* [PHP Variable Case Sensitivity](https://www.php.net/manual/en/language.variables.basics.php)

---

**Status:**
Bug is resolved by synchronizing the variable name in the controller and Blade template.
