# 🐛 Laravel ViteManifestNotFoundException Fix

This project encountered a **production deployment error** when using Laravel 12 + Vite on platforms like Railway, due to Laravel failing to locate the Vite manifest file.

---

## ❗ Problem

After running `npm run build`, the build output structure was as follows:

```

public/
└── build/
└── .vite/
└── manifest.json ✅

```

However, Laravel expects the manifest file here:

```

public/
└── build/
└── manifest.json ❌

```

### 📦 Error Message

```

Illuminate\Foundation\ViteManifestNotFoundException
Vite manifest not found at: /app/public/build/manifest.json

````

---

## ✅ Solution

Manually copy the manifest file from `.vite/manifest.json` to the location Laravel expects (`/build/manifest.json`) **after Vite finishes building**.

---

### 🔧 Step-by-Step Fix

#### 1. Add `copy-manifest.js` to project root

```js
// copy-manifest.js
import fs from 'fs';
import path from 'path';

const from = path.resolve('public/build/.vite/manifest.json');
const to = path.resolve('public/build/manifest.json');

try {
    fs.copyFileSync(from, to);
    console.log('✅ Manifest copied to public/build/manifest.json');
} catch (err) {
    console.error('❌ Failed to copy manifest.json:', err.message);
}
````

> For Node.js CommonJS, replace `import` with `require()` if needed.

---

#### 2. Update `package.json` build script

```json
"scripts": {
  "dev": "vite",
  "build": "vite build && node copy-manifest.js"
}
```

---

#### 3. Build the project

```bash
npm run build
```

---

## 📘 Why This Happens

In newer versions of **Vite (v5+)**, the `manifest.json` is moved into `.vite/` by default for better organization. However, **Laravel still looks for it in the legacy path**. Until Laravel officially supports this new structure, manual copying is required.

---

## 🚀 Optional (CI/CD)

For production deployment platforms like Railway or Render, add this to your build or postbuild steps:

```bash
npm install
npm run build
```

---

## 📦 Final Folder Structure (after build)

```
public/
  └── build/
      ├── manifest.json ✅ ← Laravel can now find this
      ├── assets/
      └── .vite/
          └── manifest.json
```

---

## 🧠 Credits

Special thanks to [Reltroner Studio](https://reltroner.com) for debugging and resolving this build integration issue with Laravel 12 + Vite + Railway.

---

Let Astralis light the unknown ✊✨
