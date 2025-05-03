# 🧪 Slug Validation Script – Chalk v5 Compatibility Issue

This README explains the issue you may encounter when using the latest version of [`chalk`](https://www.npmjs.com/package/chalk) (v5+) in your Node.js scripts, especially when validating routes like `/[slug]` pages for a static or hybrid Next.js site.

---

## ❌ Problem: `chalk.green is not a function`

### 🧾 Error Output

```bash
TypeError: chalk.green is not a function
```

This error happens when using:

```js
const chalk = require('chalk');
console.log(chalk.green("text"));
```

in **Chalk v5+**, which no longer supports CommonJS-style imports and direct usage of `chalk.green()`.

---

## 🔍 Root Cause

Chalk v5+ is **ESM-only** (ECMAScript Modules). It uses a new **instance-based API** and no longer supports the `require()` syntax or CommonJS function-style usage like `chalk.green()`.

---

## ✅ Solution 1: Use ESM Imports (Recommended for Chalk v5+)

1. **Rename your file to `.mjs`** so it supports ESM syntax:

   ```bash
   mv check-all-slugs.js check-all-slugs.mjs
   ```

2. **Use modern imports at the top:**

   ```js
   import fs from 'fs';
   import path from 'path';
   import axios from 'axios';
   import chalk from 'chalk';
   ```

3. **Now you can use:**

   ```js
   console.log(chalk.green('\n✔️ Finished! Report saved in "slug-check-report.txt"'));
   ```

4. **Run your script using Node.js:**

   ```bash
   node check-all-slugs.mjs
   ```

---

## ✅ Solution 2: Downgrade Chalk to v4 (if you want to keep `.js` and CommonJS)

If you prefer to continue using `.js` files and `require()`, downgrade chalk:

```bash
npm install chalk@4
```

Then this works fine:

```js
const chalk = require("chalk");
console.log(chalk.green("Success"));
```

---

## ℹ️ Why Did Chalk Change?

The Chalk library embraced **modern JavaScript module standards** starting from version 5. By dropping support for CommonJS (`require`), Chalk aligns with the broader **ESM-only** trend across Node.js libraries for better performance and future compatibility.
