# [BUG] Odoo 18.0 Fails to Start Windows Service and Ignores `odoo.conf` Configuration

## Summary

When installing and running Odoo 18.0 on Windows (installer version `20250529`), the service `odoo-server-18.0` fails to start correctly and continues to use the default database user `openpg` despite updating `odoo.conf` with the correct `db_user` and `db_password`.

Manual execution via CLI also fails unless security bypass flags are provided.

---

## Environment

- **OS**: Windows 11 (64-bit)
- **Odoo Version**: 18.0-20250529 (official Windows installer)
- **PostgreSQL**: 17 (installed manually)
- **Python**: bundled with Odoo (inside `Odoo/python/`)
- **Installation Type**: Executable installer

---

## Steps to Reproduce

1. Install PostgreSQL 17 manually and create user `postgres` with password `qwerty`.
2. Install Odoo 18.0 using official `.exe` installer.
3. Modify `C:\Program Files\Odoo 18.0.20250529\server\odoo.conf`:
    ```ini
    [options]
    admin_passwd = your-master-password
    db_host = localhost
    db_port = 5432
    db_user = postgres
    db_password = qwerty
    logfile = C:\odoo18.log
    addons_path = C:\Program Files\Odoo 18.0.20250529\server\odoo\addons
    ```
4. Restart service via `services.msc`.

---

## Expected Behavior

Odoo should read `odoo.conf`, connect to PostgreSQL using `postgres:qwerty`, and allow creating new databases via `localhost:8069`.

---

## Actual Behavior

- The service `odoo-server-18.0` fails to start with:
````

Windows could not start the odoo-server-18.0 service on Local Computer.
The service did not return an error.

````
- When running manually via:
```bash
"C:\Program Files\Odoo 18.0.20250529\python\python.exe" odoo-bin -c odoo.conf
````

It shows:

```
Using the database user 'postgres' is a security risk, aborting.
```

* The logs also show:

  ```
  odoo: database: openpg@localhost:5432
  ```

  despite setting `db_user = postgres` in the configuration.

---

## Root Cause

* The Odoo Windows service appears to:

  * Ignore `odoo.conf` or fallback to internal default config.
  * Enforce security policy rejecting usage of `postgres` user without explicit override.
  * Cache internal connection config from installation time (`openpg`).

---

## Workaround

### ✅ Manual CLI Run with Override:

```bash
cd "C:\Program Files\Odoo 18.0.20250529\server"
"C:\Program Files\Odoo 18.0.20250529\python\python.exe" odoo-bin -c odoo.conf --db-user=postgres --db-password=qwerty
```

### ✅ Alternative: Create a dedicated PostgreSQL role

1. Create new PostgreSQL user:

   ```sql
   CREATE ROLE odoo WITH LOGIN PASSWORD 'reltroner123';
   ALTER ROLE odoo CREATEDB;
   ```
2. Update `odoo.conf`:

   ```ini
   db_user = odoo
   db_password = reltroner123
   ```

---

## Suggested Fix

* Ensure `odoo.conf` is respected by the Windows service by passing its path explicitly during service registration or making its usage persistent.
* Allow `postgres` user for development environments with a safety warning instead of full abortion.
* Improve error messaging and fallback behavior when config conflicts occur.

---

## Tags

`#odoo18` `#windows-service` `#psycopg2` `#postgres` `#odoo-bin` `#service-failure` `#odoo.conf` `#authentication-failed`
