# 🚨 Keycloak Admin UI Bug on Railway Deployment

## Issue Summary

After deploying **Keycloak 24.0.1** using a Docker container to Railway and accessing the admin UI at:

```

[https://sso.reltroner.com/admin/master/console/](https://sso.reltroner.com/admin/master/console/)

```

The browser gets stuck on:

> **"Loading the Admin UI"**

While the backend is successfully running and reachable via:

```

[https://sso.reltroner.com/realms/master](https://sso.reltroner.com/realms/master)

````

and returns the expected JSON configuration.

---

## Deployment Context

- Platform: [Railway](https://railway.app)
- Docker Base Image: `quay.io/keycloak/keycloak:24.0.1`
- Deployment Command:
  ```bash
  /opt/keycloak/bin/kc.sh start --hostname sso.reltroner.com --hostname-strict=false --hostname-strict-https=false --proxy edge
````

* Domain: `sso.reltroner.com` (CNAME properly set via Namecheap)

---

## Root Cause

The Admin UI is a React-based frontend that:

* Expects a valid `hostname` setting in sync with the public domain.
* Fails to load when behind proxies unless `--proxy edge` is correctly applied.
* Can be blocked by **CDN caching**, **CORS errors**, or **JS routing mismatch**.

---

## Resolution Steps

### ✅ 1. Ensure correct container start command

```bash
/opt/keycloak/bin/kc.sh start --hostname sso.reltroner.com --hostname-strict=false --hostname-strict-https=false --proxy edge
```

### ✅ 2. Confirm the JSON realm endpoint is reachable:

Visit:

```
https://sso.reltroner.com/realms/master
```

Expected output:

```json
{
  "realm": "master",
  "public_key": "...",
  "token-service": "...",
  ...
}
```

### 🔁 3. Clear browser cache / Hard refresh

Use:

* `Ctrl + Shift + R`
* Or try Incognito/Private mode
* Or different browser (e.g. Firefox)

### 🧪 4. Debug in DevTools

Check for:

* Blocked JS file loading
* 404s for `/admin/resources/*`
* CORS or CSP errors

---

## Status

| Checkpoint                                  | Status       |
| ------------------------------------------- | ------------ |
| Backend reachable at `/realms/master`       | ✅            |
| Admin user created (`admin / reltroner123`) | ✅            |
| UI loads on `/admin/master/console`         | ⚠️ *Pending* |
| Proper hostname & proxy setup               | ✅            |
| Railway deployment logs clean               | ✅            |

---

## Additional Notes

This is a common issue when running **Keycloak behind proxies/CDN** or using **Railway's custom domains**. Waiting 15–30 minutes may resolve cache-related delays.

If the UI is still stuck after propagation:

* Check Network tab (DevTools) for blocked assets
* Consider setting `KEYCLOAK_FRONTEND_URL` if proxying is more complex

---

## License

MIT — © 2025 Reltroner Studio

