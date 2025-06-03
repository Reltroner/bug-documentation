# 🛠️ Bug Report: Keycloak SSO - "Page not found" on Login Redirect

## ❗ Problem

During login via **SSO (Keycloak)** on Laravel app, user gets the following error:

```

Page not found

```

This usually means that the `redirect_uri` is **not allowed** in the Keycloak client configuration for `app-reltroner`.

---

## ✅ Step-by-Step Solution

### 1. ✅ Add the Correct Redirect URI to Keycloak

Open your Keycloak Admin Panel:

```

[https://sso.reltroner.com/admin/master/console/#/realms/reltroner/clients/app-reltroner/settings](https://sso.reltroner.com/admin/master/console/#/realms/reltroner/clients/app-reltroner/settings)

```

In the **"Valid Redirect URIs"** section, make sure you add:

```

[https://app.reltroner.com/login/keycloak/callback](https://app.reltroner.com/login/keycloak/callback)

```

For local development (optional):

```

[http://localhost:8000/login/keycloak/callback](http://localhost:8000/login/keycloak/callback)

```

Wildcard usage (⚠️ not recommended for production):

```

[https://app.reltroner.com/](https://app.reltroner.com/)\*

````

> ⚠️ **Ensure there are no extra spaces** and the URI starts with `https://` if you're using a public domain.

---

### 2. ✅ Check Your Laravel `.env` Configuration

Ensure your `.env` file contains the correct credentials from Keycloak:

```env
KEYCLOAK_CLIENT_ID=app-reltroner
KEYCLOAK_CLIENT_SECRET=your-client-secret
KEYCLOAK_REDIRECT_URI=https://app.reltroner.com/login/keycloak/callback
KEYCLOAK_BASE_URL=https://sso.reltroner.com
KEYCLOAK_REALM=reltroner
````

---

### 3. ✅ Match the Redirect URI Exactly

From the browser error:

```
https://sso.reltroner.com/realms/reltroner/protocol/openid-connect/auth?client_id=app-reltroner&redirect_uri=http%3A%2F%2Fapp.reltroner.com%2Flogin%2Fkeycloak%2Fcallback
```

✅ Make sure:

* `client_id` matches exactly with your Keycloak client ID
* `redirect_uri` matches exactly with what’s in “Valid Redirect URIs”
* You use **`https://`** (not `http://`) in production setups

---

### 4. 🧪 Verify Keycloak OpenID Endpoint

To verify that Keycloak is live and Socialite-compatible, open this URL:

```bash
https://sso.reltroner.com/realms/reltroner/.well-known/openid-configuration
```

You should see a JSON response with Keycloak metadata.

---

## 🧩 Still Getting "Page not found"?

If you're still stuck:

1. Double-check the **Valid Redirect URIs** in Keycloak.
2. Open browser **DevTools → Network tab**, and copy the failed request’s full URL.
3. Check for mismatched domain, protocol (`http/https`), or typos.

---

### Need Help?

If you're unsure what’s wrong, please share:

* The full **redirect\_uri** value used by Laravel
* Screenshot of **Valid Redirect URIs** from Keycloak
* Any console/network error logs

I'll be happy to help you debug it further.

---
