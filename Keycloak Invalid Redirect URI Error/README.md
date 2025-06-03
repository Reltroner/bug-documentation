# 🛠️ Bug Report: Keycloak Invalid Redirect URI Error

## ❗ Issue Summary

When attempting to authenticate users via Keycloak SSO on the Laravel application hosted at `https://app.reltroner.com`, the following error was encountered on the Keycloak login screen:

```

Invalid parameter: redirect\_uri

````

## 📍 Context

- **SSO Provider:** Keycloak `24.0.1`
- **Client ID:** `app-reltroner`
- **Redirect URI Used:** `http://app.reltroner.com/login/keycloak/callback`
- **Laravel Version:** 10+
- **Laravel Socialite Integration:** `socialiteproviders/keycloak`

## 💥 Root Cause

The `redirect_uri` sent from Laravel Socialite to Keycloak used the `http://` protocol instead of `https://`, which was not listed as a valid redirect URI in Keycloak’s client settings. Keycloak strictly validates the `redirect_uri` against registered entries.

## ✅ Solution

### 1. Update Environment Variable

Ensure the `KEYCLOAK_REDIRECT_URI` in `.env` and/or Railway environment variables uses HTTPS:

```env
KEYCLOAK_REDIRECT_URI=https://app.reltroner.com/login/keycloak/callback
````

### 2. Fix Keycloak Client Settings

Under **Clients > app-reltroner > Login Settings**:

| Field                           | Correct Value                                       |
| ------------------------------- | --------------------------------------------------- |
| Root URL                        | `https://app.reltroner.com`                         |
| Valid Redirect URIs             | `https://app.reltroner.com/login/keycloak/callback` |
| Valid Post Logout Redirect URIs | `https://app.reltroner.com`                         |
| Web Origins                     | `https://app.reltroner.com`                         |

### 3. Save Configuration

Click **Save** in Keycloak admin panel after adjusting login settings.

### 4. Retest SSO Login

Navigate to:

```
https://app.reltroner.com/login/keycloak
```

You should be redirected properly to Keycloak and then back to Laravel after successful login.

---

## 📎 Related Logs

```
We are sorry...
Invalid parameter: redirect_uri
```

## 🧪 Tested On

* ✅ Railway-hosted Laravel app
* ✅ Keycloak realm `reltroner`
* ✅ HTTPS enforced for all routes

---

## 🔒 Recommendation

Always enforce `https://` in all SSO production environments and match registered URIs exactly as sent by the application.

