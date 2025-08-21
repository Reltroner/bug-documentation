# 🐛 Bug Report — PHP Version Mismatch when Running Artisan

This document records the bug encountered when running Laravel Artisan commands with the default PHP CLI, and the steps taken to resolve it.

---

## 1. Bug Encountered

Attempting to run `php artisan config:clear` produced a Composer platform error:

```bash
php artisan config:clear
````

**Error:**

```
Composer detected issues in your platform:

Your Composer dependencies require a PHP version ">= 8.2.0". You are running 8.1.33.
```

---

## 2. Environment Check

Check default PHP version:

```bash
php -v
```

**Result:**

```
PHP 8.1.33 (cli)
```

➡️ Problem: The server default CLI PHP version is **8.1.33**, while the project requires **>= 8.2.0**.

---

## 3. Solution — Create Aliases for PHP 8.3 and Composer

To consistently use PHP 8.3 for Laravel and Composer:

```bash
echo 'alias php83="/opt/cpanel/ea-php83/root/usr/bin/php"' >> ~/.bashrc
echo 'alias composer83="/opt/cpanel/ea-php83/root/usr/bin/php $HOME/bin/composer83"' >> ~/.bashrc
source ~/.bashrc
```

---

## 4. Verification

Check PHP 8.3:

```bash
php83 -v
```

**Result:**

```
PHP 8.3.23 (cli)
```

Check Composer bound to PHP 8.3:

```bash
composer83 -V
```

**Result:**

```
Composer version 2.8.11
PHP version 8.3.23 (/opt/cpanel/ea-php83/root/usr/bin/php)
```

---

## 5. Retry Artisan Command

Run Artisan again with PHP 8.3:

```bash
php83 artisan config:clear
```

**Output:**

```
INFO  Configuration cache cleared successfully.
```

Run key generation:

```bash
php83 artisan key:generate
```

**Output:**

```
INFO  Application key set successfully.
```

---

## ✅ Outcome

* The bug was caused by the server defaulting to PHP 8.1 CLI.
* Added aliases to force `php83` and `composer83`.
* Artisan commands now run successfully with **PHP 8.3**, resolving the Composer platform error.
