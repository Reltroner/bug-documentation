# 🛠️ Case 5 — Install Laravel Dependencies with PHP 8.3

This document describes the process of installing Laravel dependencies using Composer with PHP 8.3, after resolving the PHP version mismatch.

---

## 1. Navigate to Project Directory

```bash
cd ~/repositories/reltroner-hr-app
````

---

## 2. Run Composer Install with PHP 8.3

Execute Composer using PHP 8.3 explicitly and enable `allow_url_fopen`:

```bash
/opt/cpanel/ea-php83/root/usr/bin/php -d allow_url_fopen=On $HOME/bin/composer83 \
  install --no-dev --optimize-autoloader
```

---

## 3. Installation Process

Composer successfully downloaded and extracted required packages:

* Symfony components (Console, Routing, HttpFoundation, etc.)
* Laravel Framework `v12.13.0`
* Laravel Sanctum, Inertia.js, Ziggy
* Other PHP dependencies (Guzzle, Monolog, Carbon, etc.)

Example logs:

```
Generating optimized autoload files
> Illuminate\Foundation\ComposerScripts::postAutoloadDump
> @php artisan package:discover --ansi

INFO  Discovering packages.
  inertiajs/inertia-laravel ........................................ DONE
  laravel/sanctum .................................................. DONE
  laravel/tinker ................................................... DONE
  nesbot/carbon .................................................... DONE
  nunomaduro/termwind .............................................. DONE
  tightenco/ziggy .................................................. DONE
```

---

## 4. Verification

* All dependencies installed without errors.
* Autoload files generated.
* Laravel package discovery completed successfully.

---

## ✅ Outcome

* Laravel dependencies installed with **Composer (PHP 8.3)**.
* Project is now ready for environment setup and key generation.
