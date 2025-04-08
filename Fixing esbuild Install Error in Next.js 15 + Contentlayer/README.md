# 🧩 Fixing `npm install esbuild` Error in Next.js 15

When working with a **Next.js 15 project** that uses `contentlayer` or `mdx-bundler`, you might face an error when trying to install `esbuild`:

```bash
npm install esbuild --save-dev
```

This will likely result in:

```bash
npm ERR! ERESOLVE could not resolve
npm ERR! peer next@"^12 || ^13" from next-contentlayer@0.3.4
npm ERR! Conflicting peer dependency: next@13.x
```

---

## 🎯 Why This Happens

The error is triggered because:

- You're using **Next.js 15**, but
- `next-contentlayer@0.3.4` only officially supports **Next.js 12 and 13** as its peer dependency.
- Installing `esbuild` (or any new package) causes `npm` to re-validate **all dependencies**, including `next-contentlayer`.

---

## ✅ Solution

Force install `esbuild` while skipping peer dependency checks:

```bash
npm install esbuild --save-dev --legacy-peer-deps
```

Or, if that doesn't work:

```bash
npm install esbuild --save-dev --force
```

These commands **bypass strict peer version checks**, allowing the install to continue.

---

## 📦 What is `esbuild`?

`esbuild` is a fast bundler and transpiler used by:
- `mdx-bundler` (for processing markdown with JSX)
- Some advanced tooling inside `contentlayer`

Even though it's not installed automatically, it's required by those libraries to function properly.

---

## 💡 Tips

- ✅ Only use `--legacy-peer-deps` or `--force` if you know what you’re doing — in this case, it’s **safe and expected**.
- ✅ If you're using `pnpm` or `yarn`, they tend to handle peer dependencies more gracefully.

---

## 📌 Final Reminder

After successful installation, you can continue with:

```bash
npm run dev
```

You should no longer see the `esbuild` error when starting your dev server.

---

> Powered by [Reltroner Studio](https://www.reltroner.com), in the pursuit of reality beyond illusion ✨
