# Reltroner HR App — Migrating from Inertia React to Blade + Alpine.js (Breeze)

## 📌 Purpose
This project initially used Laravel Breeze with the **Inertia + React** stack and has been successfully migrated to the **Blade + Alpine.js** stack without deleting the entire project or database. This document details the step-by-step migration process, encountered errors, and their solutions.

---

## ✅ Migration Steps: Switching Breeze Stack

### 1. Manually remove Inertia + React stack
```bash
rm -rf resources/js
rm -rf resources/views
rm resources/views/app.blade.php
````

### 2. Create placeholder files to avoid installation failure

Create:

* `resources/views/welcome.blade.php`
* `resources/js/app.js`

### 3. Reinstall Breeze with the Blade stack:

```bash
php artisan breeze:install blade
```

> Error will occur because `resources/js/app.js` tries to import a missing `./bootstrap`.

---

## 🧨 Encountered Errors & Fixes

### Error 1: Missing `welcome.blade.php`

```bash
file_get_contents(.../welcome.blade.php): Failed to open stream: No such file or directory
```

✅ Fix:

```bash
touch resources/views/welcome.blade.php
```

---

### Error 2: Missing `resources/js/app.js`

```bash
copy(.../resources/js/app.js): Failed to open stream: No such file or directory
```

✅ Fix:

```bash
mkdir -p resources/js
echo "// dummy" > resources/js/app.js
```

---

### Error 3: Missing `bootstrap.js`

```bash
Could not resolve "./bootstrap" from "resources/js/app.js"
```

✅ Fix:
Create `resources/js/bootstrap.js` with:

```js
import Alpine from 'alpinejs';
window.Alpine = Alpine;
Alpine.start();
```

Update `app.js` to:

```js
import './bootstrap';
```

---

## 🚀 Rebuild Frontend

```bash
npm install
npm run build
```

### ✅ Successful Build Output:

```bash
vite v6.3.5 building for production...
✓ 4 modules transformed.
public/build/manifest.json             0.27 kB │ gzip:  0.15 kB
public/build/assets/app-BbaVIuwS.css  28.78 kB │ gzip:  5.65 kB
public/build/assets/app-L2ZqPtNd.js   44.37 kB │ gzip: 16.06 kB
✓ built in 994ms
```

---

## 🎉 Breeze Blade + Alpine Successfully Installed

You can now access:

* `/login`, `/register`, and `/dashboard`
* Main layout: `resources/views/layouts/app.blade.php`

---

## 🧠 Additional Tips

* Want to add Alpine.js dropdowns, modals, or toggles? Add them to `layouts/app.blade.php` or partial views.
* See the [Alpine.js documentation](https://alpinejs.dev/start-here) for more interactive UI patterns.

---

Made with 💻 by Reltroner Studio
