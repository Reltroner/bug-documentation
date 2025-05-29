# [BUG] Odoo 18.0 Fails to Create Database: "permission denied to create database"

## Summary

After successfully installing and configuring Odoo 18.0 on Windows 11 with PostgreSQL 17, attempting to create a new database via the web interface (`http://localhost:8069/web/database/create`) fails with the following error:

```

Database creation error: permission denied to create database

````

---

## Environment

- **OS**: Windows 11 Pro (64-bit)
- **Odoo Version**: 18.0-20250529
- **PostgreSQL**: 17
- **Installation Type**: Executable installer (`.exe`)
- **Database User Used**: `postgres` or a custom user defined in `odoo.conf`

---

## Steps to Reproduce

1. Install PostgreSQL 17 manually and configure user `postgres` with password.
2. Install Odoo 18.0 using the `.exe` installer.
3. Update `odoo.conf` with correct DB credentials:
   ```ini
   db_host = localhost
   db_port = 5432
   db_user = postgres
   db_password = qwerty
````

4. Launch Odoo manually:

   ```bash
   "C:\Program Files\Odoo 18.0.20250529\python\python.exe" odoo-bin -c odoo.conf
   ```
5. Open browser to `http://localhost:8069`
6. Fill the database creation form using a unique name.
7. Click **Create database**

---

## Expected Behavior

* A new database is successfully created using the configured PostgreSQL user.

---

## Actual Behavior

* Odoo fails with error:

  ```
  Database creation error: permission denied to create database
  ```

---

## Root Cause

The PostgreSQL user specified in `odoo.conf` (e.g., `postgres` or `odoo`) does **not have the `CREATEDB` privilege**, preventing Odoo from programmatically creating a new database.

---

## Solution / Workaround

### ✅ Option 1 — Grant `CREATEDB` to existing user:

Using SQL:

```sql
ALTER ROLE postgres CREATEDB;
```

Or using pgAdmin:

1. Open `pgAdmin`
2. Navigate to **Login/Group Roles > postgres**
3. Right-click > **Properties**
4. Under **Privileges** tab, enable:

   * ✅ Can create databases
5. Save and restart Odoo

---

## Recommendation

Odoo should detect and provide clearer guidance when the PostgreSQL user lacks `CREATEDB` permissions. Alternatively, documentation should explicitly mention this prerequisite for first-time installations.

---

## Tags

`#odoo18` `#postgresql` `#createdb` `#permission-error` `#database-creation` `#windows-installation` `#odoo.conf`

