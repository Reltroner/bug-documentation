# 🐞 Bug Documentation: JSON Output Instead of Blade View

## Description

When accessing the `/accounts` page in the browser, the page displays raw JSON data instead of rendering the intended Blade view. This issue causes the UI to be unusable for end users expecting an HTML table or styled component.

## Root Cause

The controller responsible for the `/accounts` route (`AccountController@index`) returns a JSON response by default, rather than a Blade view. This is typically caused by the following implementation:

```php
// app/Http/Controllers/AccountController.php

public function index(): JsonResponse
{
    $accounts = Account::with('parent', 'children', 'budgets')->paginate(20);
    return response()->json($accounts);
}
````

## Expected Behavior

When visiting `/accounts` in a web browser, the user should see an HTML page (rendered with Blade), such as a data table or list view, **not** a raw JSON string.

## Actual Behavior

The browser displays the following:

```json
{"current_page":1,"data":[{"id":1,"code":"123.45", ... }], ... }
```

## Steps to Reproduce

1. Visit `/accounts` in the browser.
2. Observe that the response is JSON, not an HTML page.

## Solution

**Refactor** the `index()` method to return a Blade view, passing the paginated data:

```php
public function index()
{
    $accounts = Account::with('parent', 'children', 'budgets')->paginate(20);
    return view('accounts.index', compact('accounts'));
}
```

## References

* [Laravel Controllers & Responses](https://laravel.com/docs/12.x/controllers)
* [Blade Templates](https://laravel.com/docs/12.x/blade)

---

**TL;DR:**
If you see JSON output in the browser instead of a Blade view, ensure your controller returns a view, not `response()->json()`.
