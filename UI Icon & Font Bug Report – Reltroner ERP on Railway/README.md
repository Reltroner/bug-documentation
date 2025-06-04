# 🐞 UI Icon & Font Bug Report – Reltroner ERP on Railway

## Overview

This project encountered an issue where icons (such as **Dripicons**) and **custom fonts** failed to render correctly after deployment to [Railway.app](https://railway.app). While all assets (CSS, JS, fonts, images) were present and accessible via HTTPS, the icons were still invisible in the live UI.

---

## ✅ Symptoms

- No icons were shown in the sidebar (e.g., logout, dashboard, etc.).
- Browser DevTools console displayed no specific error.
- Font files like `.woff`, `.ttf`, `.eot`, `.svg` were downloadable directly.
- The `<img src="{{ asset('images/logo.png') }}` displayed broken image even though the path was valid.
- Stylesheets and fonts were blocked due to mixed content (initially).

---

## 🔍 Root Causes

1. **Mixed Content Warning**  
   Some `<link>` and `<script>` tags were loading via `http://` instead of `https://`, causing browsers to block them silently under HTTPS.

2. **Font MIME Type Not Set**  
   Railway does not automatically recognize certain font types like `.woff`, `.ttf`, etc., unless explicitly declared via `.htaccess`.

3. **CORS Policy on Fonts**  
   Font requests without the proper `Access-Control-Allow-Origin` header were ignored by the browser, leading to invisible icons.

---

## ✅ Fix Steps

### 1. Force HTTPS in All Asset Links

Ensure you use:
```blade
{{ secure_asset('...') }}
````

instead of just:

```blade
{{ asset('...') }}
```

### 2. Upload All Font Files for Dripicons

Make sure the following files exist in:
`public/mazer/assets/extensions/@icon/dripicons/`

* `dripicons.css`
* `dripicons.eot`
* `dripicons.woff`
* `dripicons.ttf`
* `dripicons.svg`

### 3. Add `.htaccess` Font MIME + CORS Fix

Append the following to your `public/.htaccess`:

```apache
<IfModule mod_mime.c>
  AddType application/vnd.ms-fontobject .eot
  AddType font/ttf .ttf
  AddType font/woff .woff
  AddType font/woff2 .woff2
  AddType image/svg+xml .svg
</IfModule>

<IfModule mod_headers.c>
  <FilesMatch "\.(ttf|ttc|otf|eot|woff|woff2|font.css|css|svg)$">
    Header set Access-Control-Allow-Origin "*"
  </FilesMatch>
</IfModule>
```

---

## ✅ Result

After fixing `.htaccess`, updating to `secure_asset`, and deploying:

* ✅ All icons and logos rendered correctly
* ✅ Fonts loaded with correct MIME types and no CORS issues
* ✅ Visual consistency restored in production

---

## 🧠 Lessons Learned

* Always validate HTTPS links on production
* Explicitly configure font MIME types and CORS when hosting on Railway or other PaaS providers
* Use browser DevTools > Network tab > "Fonts" to debug missing icons

---

## 🙌 Credits

Crafted with ❤️ by [Rei Reltroner](https://www.reltroner.com)
Reltroner ERP & SSO – 2025
