# 🔐 Keycloak Initial User Setup (Reltroner Realm)

This guide documents the manual steps to create the initial admin or test user (`reltroner`) in the **`reltroner`** realm of your Keycloak instance at [`sso.reltroner.com`](https://sso.reltroner.com).

---

## 🚫 Problem

When accessing your Keycloak login page via `/login/keycloak`, you may encounter this error:

```

Invalid username or password

```

This happens because **no user has been created** in the `reltroner` realm yet.

---

## ✅ Solution: Create Initial User (`reltroner`)

### 1. Log In to Keycloak Admin Console

- URL: [https://sso.reltroner.com/admin/](https://sso.reltroner.com/admin/)
- Log in using your Keycloak admin account (created via Docker `KEYCLOAK_ADMIN` env)

---

### 2. Switch to the `reltroner` Realm

- From the realm dropdown (top-left), select: `reltroner`

---

### 3. Go to Users → Create User

- Click `Users` from the sidebar
- Click the `Create user` button
- Fill in the following:

| Field            | Value                 |
|------------------|-----------------------|
| Username         | `reltroner`           |
| Email            | `reltroner@reltroner.com` _(optional)_ |
| Email Verified   | ✅ (Enable it)         |
| Enabled          | ✅ (Ensure it's active) |

Then click `Create`.

---

### 4. Set User Password

- After creating the user, open the **`Credentials`** tab
- Enter a password, e.g. `reltroner123`
- Set **Temporary = OFF**
- Click **Set Password**

---

### ✅ Now You Can Log In

- Visit: [https://app.reltroner.com/login/keycloak](https://app.reltroner.com/login/keycloak)
- Use:

```

Username: reltroner
Password: reltroner123

````

---

## 📝 Optional: Export as Realm JSON

To persist this user across container rebuilds, export the realm configuration as a `.json` and use it during container boot via `--import-realm`.

Example:

```bash
/opt/keycloak/bin/kc.sh import --file /opt/keycloak/data/import/reltroner-realm.json
````

---

## ❗ Disclaimer

If you recreated the `reltroner` realm due to a container reset or volume loss, **all clients and users will be deleted** unless you import them from a backup.

---

For more setup automation and persistent Keycloak state management, feel free to open a discussion or contribute to this repo.

—
Reltroner Studio 🚀
