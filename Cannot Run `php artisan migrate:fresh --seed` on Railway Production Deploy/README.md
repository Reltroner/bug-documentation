# 🚩 Bug: Cannot Run `php artisan migrate:fresh --seed` on Railway Production Deploy

## Description

When deploying this Laravel project to [Railway](https://railway.app), setting the **Pre-deploy Command** to:

```sh
php artisan migrate:fresh --seed
````

results in a failed deployment with the following warning in the Railway build logs:

```
APPLICATION IN PRODUCTION.

WARN  Command cancelled.
```

The deployment stops, and migrations/seeding are **not executed**.

---

## Root Cause

Laravel protects destructive commands such as `migrate:fresh` in production environments by requiring user confirmation.
In non-interactive CI/CD environments like Railway, this prompt **cannot be answered**, causing the command to be cancelled automatically.

---

## Solution

Add the `--force` flag to the artisan command, so it skips the confirmation prompt and runs non-interactively:

```sh
php artisan migrate:fresh --seed --force
```

Update your Railway project's **Pre-deploy Command** to:

```
php artisan migrate:fresh --seed --force
```

Your **Start Command** can remain unchanged:

```
php -S 0.0.0.0:$PORT -t public
```

---

## ⚠️ Warning

* `php artisan migrate:fresh --seed --force` **drops all tables, migrates, and re-seeds the database** on every deployment.
* **Never use this in production** environments where you have real, persistent data.
* Use only for staging, testing, or demo deployments.

---

## References

* [Laravel Docs: Artisan Console - `--force` flag](https://laravel.com/docs/artisan#forcing-commands-to-run-in-production)
* [Railway Docs: Pre-Deploy Command](https://docs.railway.app/develop/cli#predeploy)

---

## Example Railway Deployment Configuration

| Setting            | Value                                      |
| ------------------ | ------------------------------------------ |
| Pre-deploy command | `php artisan migrate:fresh --seed --force` |
| Start command      | `php -S 0.0.0.0:$PORT -t public`           |

---

**If you see `APPLICATION IN PRODUCTION. WARN  Command cancelled.` in the logs, always check for missing `--force` on artisan commands!**
