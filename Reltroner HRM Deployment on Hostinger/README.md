# Reltroner HRM Deployment on Hostinger

This document records the deployment steps of **Reltroner HRM** (Laravel 12) into Hostinger shared hosting environment after domain transfer from Namecheap.

---

## 📦 Composer Setup

On Hostinger shared hosting, the default `composer` binary is Composer 1.x, which is not compatible with Laravel 12 (requires Composer 2).  
We solved this by installing Composer 2 locally inside the project directory.

```bash
cd ~/domains/reltroner.com/public_html/hrm

# Download installer
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

# Install Composer 2 as local binary
php composer-setup.php --2 --filename=composer.phar

# Cleanup
php -r "unlink('composer-setup.php');"

# Verify
php composer.phar -V
# Composer version 2.x should be displayed
````

---

## 📥 Installing Dependencies

Since Laravel 12 requires Composer 2, always run Composer via the local binary:

```bash
cd ~/domains/reltroner.com/public_html/hrm

# Install dependencies (production only)
php composer.phar install --no-dev --optimize-autoloader

# If memory limit issue occurs:
php -d memory_limit=-1 composer.phar install --no-dev --optimize-autoloader
```

Optional: create an alias for convenience during the session:

```bash
alias composer="php ~/domains/reltroner.com/public_html/hrm/composer.phar"
composer -V   # must output version 2.x
```

---

## ⚙️ Laravel Configuration

1. Copy and edit environment file:

```bash
[ -f .env ] || cp .env.example .env
nano .env
```

Minimum required configuration:

```dotenv
APP_ENV=production
APP_DEBUG=false
APP_URL=https://hrm.reltroner.com

DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=<your_hostinger_db>
DB_USERNAME=<your_db_user>
DB_PASSWORD=<your_db_password>
```

2. Generate application key and run migrations:

```bash
php artisan key:generate --force
php artisan migrate --force
```

3. Optimize and set permissions:

```bash
php artisan config:cache
php artisan route:clear      # safer than route:cache at this stage
php artisan view:cache

chmod -R 775 storage bootstrap/cache
find public -type d -exec chmod 755 {} \;
find public -type f -exec chmod 644 {} \;
```

---

## 🔍 Health Check

Add a quick test file:

```bash
echo "<?php echo 'OK';" > public/health.php
```

Then visit:
➡️ `https://hrm.reltroner.com/health.php`

(Remove the file after testing.)

---

## 🌐 DNS & SSL

1. **DNS Zone (hPanel):**

   * Add A record for `hrm` → Hostinger server IP (e.g. `145.79.28.61`).

2. **SSL (hPanel → Security → SSL):**

   * Enable free Let’s Encrypt SSL for `hrm.reltroner.com`.

---

## ✅ Summary

* Composer 2 installed locally (`composer.phar`).
* Dependencies installed successfully.
* `.env` configured for production.
* Migrations and caches executed.
* Permissions fixed.
* Domain DNS pointed correctly to Hostinger.
* SSL enabled for subdomain.

At this point, the Laravel HRM module should be accessible at:

👉 `https://hrm.reltroner.com`

