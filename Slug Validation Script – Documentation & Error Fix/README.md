# 🛠 Slug Validation Script – Documentation & Error Fix

## Overview

This script, `check-all-slugs.mjs`, automatically validates all `[slug]` routes in a Next.js content-driven site by crawling the `content/<category>/*.md` folders and testing the rendered pages on `https://www.reltroner.com`.

---

## ❌ Error Encountered

### ❗️Error Message

```
SyntaxError: Identifier 'fs' has already been declared
```

### 📍Stack Trace

```
const fs = require('fs');
      ^

SyntaxError: Identifier 'fs' has already been declared
    at compileSourceTextModule (...)
```

### 📦 Root Cause

This error occurred because the script mixed two module systems:

* `import fs from 'fs';` → ES Modules (ESM)
* `const fs = require('fs');` → CommonJS

Node.js does not allow combining ESM (`import`) and CommonJS (`require`) syntax in `.mjs` files. Attempting to declare the same module twice causes a syntax error.

---

## ✅ Resolution

### 🔄 Fix Applied

1. **Removed all `require(...)` declarations.**
2. **Used only ES Module (`import`) syntax** at the top of the file.

### ✅ Corrected Import Section

```js
import fs from 'fs';
import path from 'path';
import axios from 'axios';
import chalk from 'chalk';
```

---

## 🧪 How to Run the Script

1. Make sure you have Node.js (v18+).
2. Install the dependencies:

```bash
npm install chalk axios
```

3. Run the script:

```bash
node check-all-slugs.mjs
```

---

## 🗂 Output

The script will:

* Read all markdown slugs under `/content/<category>/`
* Ping each route at `https://www.reltroner.com/<category>/<slug>`
* Write results to a log file: `slug-check-report.txt`

Each line will be labeled with:

* ✅ Success (HTTP 200 OK)
* ⚠️ Client-side render issues
* ❌ Failed routes (404, 500, etc.)

---

## 📌 Note

> If using `.js` extension instead of `.mjs`, ensure your `package.json` includes:

```json
"type": "module"
```
