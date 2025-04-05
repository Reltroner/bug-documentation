---
title: "Fixing Hydration Mismatch and Layout Shift on Small Screens"
description: "Documentation of a bug related to hydration mismatch and layout padding on iPhone 4 viewport"
date: "2025-04-05"
author: "Rei Reltroner"
---

# 🐛 Bug Summary
A hydration mismatch and layout issue occurred on mobile devices, particularly on **iPhone 4 (320px width)**. The root of the issue was:

- Hydration mismatch error during server-side rendering due to browser extension interference and DOM differences.
- UI content appeared too **left-aligned** or **mepet ke kiri** on small screens due to missing responsive padding.

---

## ⚠️ Error Message (Hydration Mismatch)
```
Error: A tree hydrated but some attributes of the server rendered HTML didn't match the client properties...
https://react.dev/link/hydration-mismatch
```

---

## 🔍 Root Cause

### 1. **Hydration Mismatch:**
- Grammarly browser extension inserted dynamic `data-*` attributes into `<body>` which did not exist on the server-rendered HTML.
- This caused React's hydration to fail with mismatch errors.

### 2. **Layout Shift on Mobile:**
- The root layout component (`layout.jsx`) did not include any horizontal padding (`px-4`) or responsive container logic.
- On small screen widths (like iPhone 4), this caused components like images and paragraphs to appear pressed against the left edge.

---

## 🛠️ Solution

### ✅ Hydration Error Fix
- Disable browser extensions like Grammarly during development.
- Avoid using `Date.now()`, `Math.random()`, or `window` in server-rendered code.
- Wrap layout or page content in a safe hydration boundary if needed:

```tsx
'use client'
import { useEffect, useState } from 'react';

export default function HydrationSafe({ children }) {
  const [hydrated, setHydrated] = useState(false);
  useEffect(() => setHydrated(true), []);
  return hydrated ? <>{children}</> : null;
}
```

### ✅ Layout Fix for Small Screens
Updated the layout component as follows:

```tsx
export default function CultureLayout({ children }) {
  return (
    <div className="px-4 py-6 bg-slate-100 min-h-screen">
      <div className="max-w-screen-md mx-auto">
        {children}
      </div>
    </div>
  );
}
```

- `px-4` ensures safe horizontal spacing on mobile
- `max-w-screen-md` + `mx-auto` centers content responsively
- `min-h-screen` maintains full height visual consistency

---

## ✅ Status

- ✅ Bug Fixed on: **2025-04-05**
- ✅ Verified on: Desktop + iPhone 4 emulation
- 🔒 Monitored for future layout shift or hydration inconsistency

---

## 🧠 Lessons Learned

- Extensions like Grammarly can interfere with hydration — always test on a clean browser.
- Never assume SSR output is consistent on client unless DOM is fully controlled.
- Always apply `px-4` or use responsive containers when targeting small screens like iPhone 4.

---

**Reltroner Studio Debug Log – Entry #HX-0425**

---
Let Astralis light the unknown
---
