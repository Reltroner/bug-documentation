# 🛠️ Fixing `npm install contentlayer next-contentlayer` Error on Next.js 15

When trying to install `contentlayer` and `next-contentlayer` for use in a Next.js 15+ project, you may encounter the following error:

```bash
npm ERR! While resolving: learn-nextjs@0.1.0
npm ERR! Found: next@15.2.4
npm ERR! Could not resolve dependency:
npm ERR! peer next@"^12 || ^13" from next-contentlayer@0.3.4
```

This happens because `next-contentlayer@0.3.4` only officially supports **Next.js 12 and 13**, while your project uses **Next.js 15**.

---

## ✅ Solution

Run the following command **with a peer-dependency override**:

```bash
npm install contentlayer next-contentlayer --legacy-peer-deps
```

This will bypass the version conflict and install both packages.

---

## 🔍 Why this Works

The `--legacy-peer-deps` flag allows NPM to skip strict dependency checks.  
It’s useful when using newer versions of frameworks (like Next.js 15) with packages that haven't updated their peer dependencies yet.

---

## ✅ After Successful Installation

Make sure to:

1. ✅ Add `withContentlayer` to your `next.config.js`:

```js
const { withContentlayer } = require("next-contentlayer");

module.exports = withContentlayer({
  reactStrictMode: true,
});
```

2. ✅ Create `contentlayer.config.ts` or `.js` for your markdown structure.

3. ✅ Run your dev server:

```bash
npm run dev
```

---

## 📦 Note for the Future

Once `next-contentlayer` officially supports Next 15, this workaround won’t be necessary.  
Until then, this method is safe and widely used by the community.

---

> 💡 **Tip:** If you're starting a new project, consider using `pnpm` or `yarn` which handle peer dependencies more gracefully.

---

Built with ❤️ by [Reltroner Studio](https://www.reltroner.com)
