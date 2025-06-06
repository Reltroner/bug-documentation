# Keycloak Logout Redirect Loop Bug (Invalid `redirect_uri`)

This document explains a known bug experienced during logout redirect using Keycloak (tested on v24.0.1), which may cause an **infinite error loop** even when configurations are correct.

---

## ❗ Problem

When a user logs out via:

```

[https://sso.reltroner.com/realms/reltroner/protocol/openid-connect/logout?redirect\_uri=https://app.reltroner.com](https://sso.reltroner.com/realms/reltroner/protocol/openid-connect/logout?redirect_uri=https://app.reltroner.com)

```

or

```

[https://sso.reltroner.com/realms/reltroner/protocol/openid-connect/logout?redirect\_uri=https://app.reltroner.com/login/keycloak](https://sso.reltroner.com/realms/reltroner/protocol/openid-connect/logout?redirect_uri=https://app.reltroner.com/login/keycloak)

```

Keycloak returns:

```

We are sorry...
Invalid parameter: redirect\_uri

````

Even though:
- The `redirect_uri` is properly registered in the Keycloak client settings
- The application URL is reachable and secured (HTTPS)
- CORS/Web Origins settings are correct

---

## 🔍 Root Cause (Based on Investigation)

This is likely caused by **Keycloak's internal redirect URI cache** or **state mismatch**:

- Keycloak sometimes doesn't reload `Valid Post Logout Redirect URIs` immediately
- There is **no "Save + Reload Realm Cache"** feature in Keycloak Admin UI
- Restarting the container may not be enough if cache is stuck

This issue is also [reported in the Keycloak GitHub repo](https://github.com/keycloak/keycloak/discussions/17262) by others using latest versions.

---

## ✅ Temporary Workaround

Disable `redirect_uri` altogether for logout.

### Laravel Route Example:

```php
Route::get('/logout', function () {
    Auth::logout();

    $keycloakLogoutUrl = env('KEYCLOAK_LOGOUT_URL', 'https://sso.reltroner.com/realms/reltroner/protocol/openid-connect/logout');

    // No redirect_uri used here
    return redirect()->away($keycloakLogoutUrl);
})->name('keycloak.logout');
````

This will simply log the user out from Keycloak without redirecting back to the app.

---

## 🛠️ Permanent Fix (If Needed)

You may try:

1. **Delete and recreate** the affected Keycloak client (`app-reltroner`)
2. Re-add all redirect URIs:

   ```
   https://app.reltroner.com
   https://app.reltroner.com/*
   https://app.reltroner.com/login/keycloak
   ```
3. Restart Keycloak server instance
4. Ensure logout request uses the exact URL from the list above (no trailing slashes)

If the issue persists: downgrade to Keycloak v22 or v21, which are more stable for `post_logout_redirect_uri`.

---

## 🧼 Optional: Route-Based Soft Redirect

If you still want to redirect to login manually after logout:

```php
Route::get('/logout', function () {
    Auth::logout();
    return redirect('/login/keycloak'); // handled internally, avoids Keycloak redirect loop
});
```

---

## 💬 Final Notes

If you're facing this bug, **you're not alone**.
This issue has affected many developers integrating Keycloak in SPA, Laravel, and microservices apps.

Keycloak is powerful — but its logout behavior with redirect URIs can be **fragile** and inconsistent in newer releases.

---

### 🧠 You Are Not Failing — You Just Discovered a Rare Edge Case.

Let this README serve as a warning and a guide for others.
PRs and discussions welcome.

— Reltroner Studio 🧩
