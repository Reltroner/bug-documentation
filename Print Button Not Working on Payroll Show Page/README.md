# 🐛 Print Button Not Working on Payroll Show Page

## Summary

The **Print** button on the `payrolls/show.blade.php` page does not trigger the `window.print()` function, even though the JavaScript event listener is correctly defined using `@push('scripts')`.

---

## 🔍 Root Cause

The `@push('scripts')` directive is used to inject page-specific scripts. However, the parent layout `layouts.dashboard.blade.php` **did not include the corresponding `@stack('scripts')` directive**, which means the script block never renders in the final HTML output.

As a result, the following script was ignored entirely:

```blade
@push('scripts')
<script>
    document.getElementById('btnPrint').addEventListener('click', function () {
        let printContents = document.getElementById('printableArea').innerHTML;
        let originalContents = document.body.innerHTML;

        document.body.innerHTML = printContents;
        window.print();
        document.body.innerHTML = originalContents;
        window.location.reload();
    });
</script>
@endpush
````

---

## ✅ Solution

### 1. **Update Layout File**

In `resources/views/layouts/dashboard.blade.php`, ensure that the following line is added **before the closing `</body>` tag**:

```blade
@stack('scripts')
```

Example:

```blade
<!-- Other footer content... -->
@stack('scripts')
</body>
</html>
```

This will correctly render all `@push('scripts')` blocks from child views.

---

### 2. ✅ **Alternative (Direct Script Embed)**

If you prefer not to use `@push` and `@stack`, you can directly place the script at the bottom of the Blade file:

```blade
@section('content')
    <!-- Page Content -->
@endsection

<script>
    document.getElementById('btnPrint').addEventListener('click', function () {
        let printContents = document.getElementById('printableArea').innerHTML;
        let originalContents = document.body.innerHTML;

        document.body.innerHTML = printContents;
        window.print();
        document.body.innerHTML = originalContents;
        window.location.reload();
    });
</script>
```

---

## ✅ Result

* ✅ Print button now triggers `window.print()`.
* ✅ Printable content isolated using the `#printableArea` container.
* ✅ Page reloads after printing to restore original view.

---

## Related Files

* `resources/views/layouts/dashboard.blade.php`
* `resources/views/payrolls/show.blade.php`

---

> Let Astralis light the unknown. 🌌
> — Reltroner Bug Fix Archive
