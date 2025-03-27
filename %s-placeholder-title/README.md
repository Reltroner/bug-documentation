```
# `%s` Placeholder in Next.js Metadata Title

In Next.js App Router, dynamic title templates can be defined using the `%s` placeholder in the `metadata` object. This is especially useful for maintaining consistent branding across all pages of your application.

## ✅ Correct Usage

You can define a global `title` template inside your `app/layout.jsx` (or `layout.tsx`) like this:

```
export const metadata = {
  title: {
    default: "Reltroner Studio",
    template: "%s | Reltroner Studio",
  },
  description: "Reltroner Studio is a digital agency specializing in web development and creative sanctuary of the fictional universe Asthortera.",
};
```

- `%s` is a placeholder that will be replaced by the page-specific title.
- `default` is the fallback title when no specific title is set on a page.

## 💡 How It Works

If you define a `title` in a page's metadata, for example:

```js
// app/about/page.jsx or app/about/page.tsx
export const metadata = {
  title: "About",
};
```

Then the final rendered title in the browser tab will be:

```
About | Reltroner Studio
```

This happens because `%s` in the template is replaced with `"About"`.

## ❌ Common Mistake

Do **not** write:

```js
template: "s% | Reltroner Studio"
```

This will render literally as:

```
s% | Reltroner Studio
```

instead of replacing the placeholder.

## 🧠 When to Use

Use the `%s` title template if:
- You want consistent branding in the browser tab.
- You want to automate dynamic page titles.
- You want better SEO with contextual titles.

## 🔁 Nested Layouts

Each layout file (e.g., `app/blog/layout.jsx`) can override or extend the metadata template for sub-paths. You can define another `template` inside nested layouts as needed.

---

Let Astralis light the unknown.
```

---
