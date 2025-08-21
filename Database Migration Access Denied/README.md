# 🐛 Bug Report — Database Migration Access Denied

This document records the bug encountered when running Laravel database migrations and the troubleshooting steps taken.

---

## 1. Context

Environment:
- PHP: **8.3.23** (via `php83` alias)
- Composer: **2.8.11** bound to PHP 8.3
- Database: **MariaDB 10.11.13**

---

## 2. Bug Encountered

Running the migration with default PHP (8.1) failed due to Composer’s PHP requirement:

```bash
php artisan migrate --force --seed
````

**Error:**

```
Composer detected issues in your platform:
Your Composer dependencies require a PHP version ">= 8.2.0". You are running 8.1.33.
```

---

## 3. Switch to PHP 8.3

Aliased PHP 8.3 and Composer:

```bash
alias php83="/opt/cpanel/ea-php83/root/usr/bin/php"
alias composer83="/opt/cpanel/ea-php83/root/usr/bin/php $HOME/bin/composer83"
source ~/.bashrc
```

Verify versions:

```bash
php83 -v
# PHP 8.3.23

composer83 -V
# Composer version 2.8.11 / PHP version 8.3.23
```

---

## 4. Retry Migration with PHP 8.3

```bash
php83 artisan migrate --force --seed
```

**Error:**

```
SQLSTATE[HY000] [1044] Access denied for user 'ynoqpang_admin'@'localhost' 
to database 'ynoqpang_reltronerhrapp'
```

---

## 5. Database Verification

Logged in via MySQL CLI:

```bash
mysql -u ynoqpang_admin -p
```

Confirmed database exists but contains no tables:

```sql
SHOW DATABASES LIKE 'ynoqpang_reltronerhrapp';
+------------------------------------+
| Database (ynoqpang_reltronerhrapp) |
+------------------------------------+
| ynoqpang_reltronerhrapp            |
+------------------------------------+

USE ynoqpang_reltronerhrapp;
SHOW TABLES;
-- Empty set
```

---

## 6. Analysis

* Database `ynoqpang_reltronerhrapp` exists.
* User `ynoqpang_admin` can log in and access the database manually.
* Laravel migration fails due to **access denied** error from Artisan:

  * Possible incorrect credentials in `.env`.
  * Or insufficient privileges for `ynoqpang_admin` to create tables.

---

## ✅ Next Steps

* Double-check `.env` database configuration:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=ynoqpang_reltronerhrapp
DB_USERNAME=ynoqpang_admin
DB_PASSWORD=********
```

* Ensure the MySQL user has required privileges:

```sql
GRANT ALL PRIVILEGES ON ynoqpang_reltronerhrapp.* TO 'ynoqpang_admin'@'localhost';
FLUSH PRIVILEGES;
```

* Clear config cache again:

```bash
php83 artisan config:clear
```

* Retry migration:

```bash
php83 artisan migrate --force --seed
```

---

## 🔎 Outcome (Current Status)

* Database exists but is empty.
* User login to MySQL works manually.
* Laravel migration still blocked due to access denied error when creating tables.
* Root cause is likely **privilege misconfiguration** or **mismatched .env credentials**.

