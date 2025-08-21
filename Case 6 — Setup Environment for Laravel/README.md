# 🛠️ Case 6 — Setup Environment for Laravel

This document describes the process of configuring the environment file and generating the application key for a fresh Laravel installation.

---

## 1. Copy Default Environment File

Laravel provides a default environment template (`.env.example`). Copy it to `.env`:

```bash
cp .env.example .env
````

**Result:**

* A new `.env` file is created at the project root.
* This file will be used to configure application settings (database, mail, cache, etc.).

---

## 2. Generate Application Key

Run the following command with PHP 8.3 to generate a unique application key:

```bash
/opt/cpanel/ea-php83/root/usr/bin/php artisan key:generate
```

**Output:**

```
INFO  Application key set successfully.
```

---

## 3. Purpose of the Application Key

* Ensures encrypted data (sessions, cookies, etc.) are secured.
* Must be set before the application can run properly.
* Stored in the `.env` file as:

```
APP_KEY=base64:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

---

## ✅ Outcome

* `.env` file successfully created from template.
* Application key generated and stored.
* Laravel application is now ready for further configuration (database, cache, queue, etc.).
