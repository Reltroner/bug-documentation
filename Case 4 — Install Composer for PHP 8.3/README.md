# 🛠️ Case 4 — Install Composer for PHP 8.3

This document describes the process of installing a dedicated Composer binary that runs with PHP 8.3, since the default system Composer was bound to PHP 8.1.

---

## 1. Download Composer Installer

From the home directory:

```bash
curl -sS https://getcomposer.org/installer -o composer-setup.php
````

Verify the file:

```bash
ls -l composer-setup.php
```

**Expected Output:**

```
-rw-rw-r-- 1 ynoqpang ynoqpang 58444 Aug 21 21:36 composer-setup.php
```

---

## 2. Prepare Local Bin Directory

Ensure `$HOME/bin` exists:

```bash
mkdir -p $HOME/bin
```

---

## 3. Run Installer with PHP 8.3

Execute the installer explicitly with **PHP 8.3** and enable `allow_url_fopen`:

```bash
/opt/cpanel/ea-php83/root/usr/bin/php -d allow_url_fopen=On $HOME/composer-setup.php \
  --install-dir=$HOME/bin --filename=composer83
```

---

## 4. Successful Installation

**Output:**

```
All settings correct for using Composer
Downloading...

Composer (version 2.8.11) successfully installed to: /home/ynoqpang/bin/composer83
Use it: php /home/ynoqpang/bin/composer83
```

---

## 5. Verify Installation

Check version:

```bash
$HOME/bin/composer83 -V
```

**Result:**

```
Composer version 2.8.11 2025-08-21 11:29:39
PHP version 8.1.33 (/opt/alt/php81/usr/bin/php)
```

⚠️ **Note:** Running the binary directly binds to PHP 8.1.
➡️ Always execute Composer using PHP 8.3 explicitly:

```bash
/opt/cpanel/ea-php83/root/usr/bin/php -d allow_url_fopen=On $HOME/bin/composer83 ...
```

---

## ✅ Outcome

* Composer successfully installed as `composer83` in `~/bin`.
* Installation bound to PHP 8.3 for Laravel compatibility.
* Must run Composer via **php83** wrapper to avoid PHP version mismatch.
