# 🛠️ Google Analytics Integration & `layout.jsx` Fix (Next.js App Router)

This project encountered a common but critical issue during Google Analytics integration inside a Next.js App Router layout file. This README documents the root cause, the runtime error, and how it was properly resolved.

---

## ❌ The Error

When trying to use Google Analytics via `<Script>` inside `app/layout.jsx`, the following runtime error occurred:

```bash
Error: You are attempting to export "metadata" from a component marked with "use client", which is disallowed.
```

Additional log:
```bash
GET /factions 500
Fast Refresh had to perform a full reload due to a runtime error.
```

---

## 🔍 Root Cause

In Next.js App Router (`app/` directory):

- The file `app/layout.jsx` is a **Server Component by default**
- It is **not allowed to be marked** with `'use client'`
- Attempting to export `metadata` from a `'use client'` component causes a fatal build/runtime error
- Additionally, inserting native `<script>` tags directly inside the `<head>` tag in `layout.jsx` **has no effect**, since `<head>` is ignored there.

---

## ✅ Solution Overview

1. **Remove `'use client'`** from `app/layout.jsx`
2. **Move all client-only code (like `next/script`) into a separate Client Component**
3. **Import that Client Component inside the layout’s `<body>`**

---

## ✅ Final Implementation

### 🔧 `app/layout.jsx` (Server Component)

```jsx
import './global.css';
import Navbar from "@/components/Navbar";
import { roboto } from "./fonts";
import MobileNavbar from "@/components/MobileNavbar";
import CommandPalette from "@/components/CommandPalette";
import GoogleAnalytics from "@/components/GoogleAnalytics"; // <- New

export const metadata = {
  title: {
    default: "Reltroner Studio",
    template: "%s | Reltroner Studio",
  },
  description: "Reltroner Studio is a digital agency specializing in web development and creative sanctuary of the fictional universe Asthortera.",
};

export default function Layout({ children }) {
  return (
    <html lang="en" className={roboto.variable}>
      <body className="bg-slate-100 text-black dark:bg-gray-900 dark:text-white">
        <GoogleAnalytics />
        <MobileNavbar />
        <header>
          <CommandPalette />
          <Navbar />
        </header>
        <main className="py-5 grow">{children}</main>
        <footer className="border-t pt-4 pb-6 text-center text-xs text-gray-500">
          © {new Date().getFullYear()} Reltroner Studio. All rights reserved.
        </footer>
      </body>
    </html>
  );
}
```

---

### 🔧 `components/GoogleAnalytics.jsx` (Client Component)

```jsx
'use client';

import Script from 'next/script';

export default function GoogleAnalytics() {
  return (
    <>
      <Script
        src="https://www.googletagmanager.com/gtag/js?id=G-VZ7QYLZPF0"
        strategy="afterInteractive"
      />
      <Script id="google-analytics" strategy="afterInteractive">
        {`
          window.dataLayer = window.dataLayer || [];
          function gtag(){dataLayer.push(arguments);}
          gtag('js', new Date());
          gtag('config', 'G-VZ7QYLZPF0');
        `}
      </Script>
    </>
  );
}
```

---

## 🧠 Summary

| Do ✅ | Don’t ❌ |
|------|----------|
| Use `next/script` inside a client component | Mark `app/layout.jsx` with `'use client'` |
| Export `metadata` from Server Component | Combine `metadata` export with client-only logic |
| Use `<Script>` with `afterInteractive` for GA | Put `<script>` directly in the `<head>` of layout |

---

## 🔗 References

- [Next.js App Router Docs](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts)
- [Next.js metadata API](https://nextjs.org/docs/app/api-reference/functions/generate-metadata)
- [Google Analytics 4 Setup](https://support.google.com/analytics/answer/10089681)

---

## 📌 Additional Notes

This fix improves compatibility with modern Next.js SSR and ensures correct tracking without breaking the build. If you’re using other tools like Hotjar, Mixpanel, or external JS widgets, always inject them via Client Components and never directly into Server Components.
