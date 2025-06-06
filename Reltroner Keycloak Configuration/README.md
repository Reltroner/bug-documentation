# 🔐 Reltroner Keycloak Configuration

This repository contains the Docker configuration and client setup instructions for running a **Keycloak Identity Provider** used in the **Reltroner ERP SSO** ecosystem.

---

## 🚀 Quickstart: Run Keycloak via Docker

```dockerfile
FROM quay.io/keycloak/keycloak:24.0.1

ENV KEYCLOAK_ADMIN=admin
ENV KEYCLOAK_ADMIN_PASSWORD=reltroner123

EXPOSE 8080

RUN /opt/keycloak/bin/kc.sh build

ENTRYPOINT ["/opt/keycloak/bin/kc.sh", "start", 
  "--hostname=sso.reltroner.com", 
  "--hostname-strict=false", 
  "--hostname-strict-https=false", 
  "--proxy=edge", 
  "--http-enabled=true", 
  "--http-port=8080", 
  "--log-level=INFO"
]
````

> ✅ This Dockerfile builds Keycloak in production mode with a hostname `sso.reltroner.com`.

---

## ⚙️ Realm Setup: `reltroner`

Create a new realm named:

```
reltroner
```

> This realm is the central authentication layer for all apps under `*.reltroner.com`.

---

## 📲 Client Setup: `app-reltroner`

1. Go to: `Clients → Create client`
2. Fill General Settings:

   * **Client Type:** `OpenID Connect`
   * **Client ID:** `app-reltroner`
3. Enable capabilities:

   * ✅ Client authentication
   * ✅ Standard flow
   * ✅ Direct access grants
4. Set login settings:

   * **Root URL:** `https://app.reltroner.com`
   * **Home URL:** `https://app.reltroner.com/dashboard`
   * **Valid Redirect URIs:**

     ```
     https://app.reltroner.com/login/keycloak/callback
     https://app.reltroner.com/*
     ```
   * **Post Logout Redirect URIs:**

     ```
     https://app.reltroner.com
     ```
   * **Web Origins:** `https://app.reltroner.com`

---

## 🔐 Laravel `.env` Configuration (SSO)

```env
KEYCLOAK_CLIENT_ID=app-reltroner
KEYCLOAK_CLIENT_SECRET=your_client_secret_here
KEYCLOAK_BASE_URL=https://sso.reltroner.com
KEYCLOAK_REALM=reltroner
KEYCLOAK_REDIRECT_URI=https://app.reltroner.com/login/keycloak/callback
KEYCLOAK_LOGOUT_URL=https://sso.reltroner.com/realms/reltroner/protocol/openid-connect/logout
```

---

## 🧪 Testing SSO Flow

* **Login URL Test:**

  ```
  https://sso.reltroner.com/realms/reltroner/protocol/openid-connect/auth?client_id=app-reltroner&redirect_uri=https%3A%2F%2Fapp.reltroner.com%2Flogin%2Fkeycloak%2Fcallback&scope=openid&response_type=code
  ```

* **Logout URL:**

  ```
  https://sso.reltroner.com/realms/reltroner/protocol/openid-connect/logout?redirect_uri=https%3A%2F%2Fapp.reltroner.com
  ```

> ❗ If `Invalid parameter: redirect_uri` appears, make sure the exact URI is registered in **Valid Redirect URIs** inside Keycloak client.

---

## 🐛 Common Errors & Fixes

### 🔸 `Form is not secure` warning

**Fix:** Add HTTPS middleware in Laravel:

```php
// app/Http/Middleware/ForceHttps.php
if (!$request->secure() && app()->environment('production')) {
    return redirect()->secure($request->getRequestUri());
}
```

And register it in `App\Http\Kernel`.

---

### 🔸 `Invalid parameter: redirect_uri`

**Fix:** Ensure the redirect URI:

* Matches exactly in Keycloak client settings.
* Has no trailing slashes.
* Is URL-encoded when tested manually.

---

## 📦 Status

| Service        | URL                                                                | Status |
| -------------- | ------------------------------------------------------------------ | ------ |
| Keycloak Admin | [https://sso.reltroner.com/admin](https://sso.reltroner.com/admin) | ✅      |
| SSO Login      | [https://app.reltroner.com/login](https://app.reltroner.com/login) | ✅      |

---

## ✨ Maintained by [Rei Reltroner](https://www.reltroner.com/blog/for-recruiters)

```
Let Astralis light the unknown ✦
```
