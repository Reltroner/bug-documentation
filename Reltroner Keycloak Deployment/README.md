# 🔐 Reltroner Keycloak Deployment

This repository contains the custom Dockerfile to run a production-grade **Keycloak 24.0.1** server instance for [`sso.reltroner.com`](https://sso.reltroner.com).

---

## ⚠️ Important Disclaimer

> **Reset Warning:**
>
> If you redeploy or rebuild your Keycloak container **without restoring previous realm exports**, it will **remove all clients, roles, and configurations** including `app-reltroner`.
>
> You must **recreate all clients manually** or **automate the import via realm JSON file** to preserve authentication flow.

---

## 🐛 Bug Fix: Dockerfile ENTRYPOINT Parse Error

### ❌ Problem

Using `ENTRYPOINT` with multiple arguments in **multi-line format** caused the following error during `docker build` on Railway:

```

ERROR: failed to solve: dockerfile parse error on line 11: unknown instruction: "--hostname=sso.reltroner.com"

````

**Why?**  
Docker interprets each line as a new instruction. Multi-line `ENTRYPOINT` arrays must follow JSON array syntax **on a single line**.

---

### ✅ Solution

Use a **single-line array** format:

```dockerfile
ENTRYPOINT ["/opt/keycloak/bin/kc.sh", "start", "--hostname=sso.reltroner.com", "--hostname-strict=false", "--hostname-strict-https=false", "--proxy=edge", "--http-enabled=true", "--http-port=8080", "--log-level=INFO"]
````

---

### 💡 Alternative (not recommended for Railway)

```dockerfile
ENTRYPOINT ["/opt/keycloak/bin/kc.sh", "start"]
CMD ["--hostname=sso.reltroner.com", "--hostname-strict=false", "--hostname-strict-https=false", "--proxy=edge", "--http-enabled=true", "--http-port=8080", "--log-level=INFO"]
```

> ⚠️ In Railway, the `CMD` instruction is often ignored or overridden, causing Keycloak to fail silently.

---

## ✅ Verified Dockerfile

```dockerfile
FROM quay.io/keycloak/keycloak:24.0.1

ENV KEYCLOAK_ADMIN=admin
ENV KEYCLOAK_ADMIN_PASSWORD=reltroner123

EXPOSE 8080

RUN /opt/keycloak/bin/kc.sh build

ENTRYPOINT ["/opt/keycloak/bin/kc.sh", "start", "--hostname=sso.reltroner.com", "--hostname-strict=false", "--hostname-strict-https=false", "--proxy=edge", "--http-enabled=true", "--http-port=8080", "--log-level=INFO"]
```

---

## 📌 Deployment Notes

* Port `8080` must be exposed and available for Railway or Docker to bind
* The `--hostname` parameter must match your DNS (`sso.reltroner.com`)
* Use `--proxy=edge` if you're behind a reverse proxy like Railway or Vercel
* SSL should be managed via Railway or your DNS/CDN layer

---

## 📦 Optional: Realm Import

To avoid losing clients (e.g. `app-reltroner`), export your realm config:

```bash
./kc.sh export --dir /opt/keycloak/data/import --realm=reltroner --users=skip
```

And add this in `ENTRYPOINT` before startup:

```bash
--import-realm
```

You may also mount `realm-export.json` into your container at `/opt/keycloak/data/import`.

---

## 🙋‍♂️ Support

If you encounter errors like `Invalid parameter: redirect_uri` or need help automating realm creation, feel free to [open an issue](https://github.com/Reltroner/reltroner-keycloak/issues).

—

Crafted with ☕ by [Reltroner Studio](https://reltroner.com)
`Let Astralis light the unknown ✦`
