# Reltroner Keycloak Deployment

This repository contains the custom Dockerfile to run a production-grade Keycloak 24.0.1 server instance for [`sso.reltroner.com`](https://sso.reltroner.com).

---

## ⚠️ Bug Fix: Dockerfile ENTRYPOINT Parse Error

### ❌ Problem

Using `ENTRYPOINT` with multiple arguments written in **multi-line array format** caused the following error during `docker build` on Railway:

```

ERROR: failed to solve: dockerfile parse error on line 11: unknown instruction: "--hostname=sso.reltroner.com"

````

**Why?**  
Docker interprets each new line as a potential new instruction. If not formatted correctly, Docker thinks `--hostname=...` is an invalid command instead of an argument.

---

### ✅ Solution

Use **a single-line array** for `ENTRYPOINT` without line breaks:

```dockerfile
ENTRYPOINT ["/opt/keycloak/bin/kc.sh", "start", "--hostname=sso.reltroner.com", "--hostname-strict=false", "--hostname-strict-https=false", "--proxy=edge", "--http-enabled=true", "--http-port=8080", "--log-level=INFO"]
````

This ensures all parameters are treated as part of the same command array.

---

### 💡 Alternative (not recommended for Railway)

You may also split `ENTRYPOINT` and `CMD`:

```dockerfile
ENTRYPOINT ["/opt/keycloak/bin/kc.sh", "start"]
CMD ["--hostname=sso.reltroner.com", "--hostname-strict=false", "--hostname-strict-https=false", "--proxy=edge", "--http-enabled=true", "--http-port=8080", "--log-level=INFO"]
```

However, this approach often fails or is overridden in Railway deployments.

---

## ✅ Verified Dockerfile Example

```dockerfile
FROM quay.io/keycloak/keycloak:24.0.1

ENV KEYCLOAK_ADMIN=admin
ENV KEYCLOAK_ADMIN_PASSWORD=reltroner123

EXPOSE 8080

RUN /opt/keycloak/bin/kc.sh build

ENTRYPOINT ["/opt/keycloak/bin/kc.sh", "start", "--hostname=sso.reltroner.com", "--hostname-strict=false", "--hostname-strict-https=false", "--proxy=edge", "--http-enabled=true", "--http-port=8080", "--log-level=INFO"]
```

---

## 🔒 Notes

* Make sure port `8080` is exposed in your Railway or Docker environment
* The hostname (`--hostname=sso.reltroner.com`) must match your DNS and SSL certificate

---

For questions or setup automation (e.g. auto-import `realm.json`), feel free to open an issue or discussion.

—
Reltroner Studio 🚀
