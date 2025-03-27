# Fixing "Right Edge Sticking" Bug in Horizontal Navbar (`Disclaimer` Link Sticking)

When building a horizontally scrollable navbar using Tailwind CSS in Next.js, you might encounter a visual issue where the last navigation link (e.g., `Disclaimer`) appears **too close to the right edge** of the screen — especially on mobile or small viewports.

---

## 🐞 The Bug

Despite applying padding to the outer container (`div`) or margin to the last `<li>`, the final link still looks like it's "sticking" or "touching" the edge of the screen.

### ❌ Attempted Fixes That Didn't Work:
- Adding `pr-8` to the `<div class="overflow-x-auto">`
- Adding `mr-4` to the `<li>` element of `Disclaimer`
- Applying `pr-*` to `<ul>` without changing layout logic

---

## ✅ The Root Cause

In Tailwind CSS, a flex container (`ul.flex`) will **shrink to fit** the content width by default. This means:
- The `padding-right` on the `<ul>` has **no effect** visually unless the container stretches beyond the content.
- The layout won't reserve space to the right **unless forced** to grow.

---

## 🧪 Final & Correct Solution

Add the following two utility classes to the `<ul>` tag:

```html
<ul className="flex min-w-max gap-5 font-roboto text-base sm:text-lg pr-4">
```

### 🔍 What These Do:
- `min-w-max`: Forces the `<ul>` to take the minimum width of its entire content, preventing it from shrinking too tightly.
- `pr-4`: Applies right padding that now actually has visible effect due to `min-w-max`.

---

## ✅ Working `Navbar.jsx` Example

```jsx
import Link from "next/link";

export default function Navbar() {
    return (
        <div className="overflow-x-auto whitespace-nowrap px-4 py-3 bg-white shadow-sm">
            <nav>
                <ul className="flex min-w-max gap-5 font-roboto text-base sm:text-lg pr-4">
                    <li><Link href="/" className="hover:underline">Home</Link></li>
                    <li><Link href="/about" className="hover:underline">About</Link></li>
                    <li><Link href="/blog" className="hover:underline">Blog</Link></li>
                    <li><Link href="/contact" className="hover:underline">Contact</Link></li>
                    <li><Link href="/blog/worldbuilding" className="hover:underline">Basic</Link></li>
                    <li><Link href="/blog/disclaimer" className="hover:underline">Disclaimer</Link></li>
                </ul>
            </nav>
        </div>
    );
}
```

---

## ✅ Result

- The `Disclaimer` link no longer sticks to the right edge.
- Padding is respected due to `min-w-max` logic.
- Horizontal scroll still works on small screens.

---

## 💡 Pro Tip

This fix is especially useful when:
- Your navbar is scrollable (`overflow-x-auto`)
- You want symmetrical spacing for the first and last links
- You want to avoid hard-coding margin on specific list items

---

Let Astralis light the layout ⚡

---
