# 🐞 Mobile UI Bug Report: Code Block Overflow

## Overview

A visual UI bug was observed on mobile devices where code blocks rendered via Markdown (e.g., triple backtick \`\`\` blocks with SQL code) appear **too close to the left and right edges** of the screen. This occurs even when using Tailwind CSS with Typography plugins or standard utility classes.

This README documents the issue, its symptoms, reproduction steps, and the applied fix for clarity, consistency, and collaboration.

---

## 🔍 Bug Summary

* **Component Affected**: `<pre>` and `<code>` blocks inside `.prose`
* **Devices Affected**: Mobile devices (`max-width: 640px`)
* **Symptoms**:

  * Code blocks rendered with markdown appear with **zero horizontal padding**
  * Text inside code blocks feels **cramped and clipped** visually
  * No margin or padding between the content and screen edges

---

## 📸 Screenshot Example

> ![mobile-codeblock-overflow](./mobile-codeblock-overflow.webp)

The screenshot clearly shows SQL code blocks rendered edge-to-edge on mobile, breaking visual balance and readability.

---

## 🧪 Reproduction Steps

1. Open any markdown-rendered page with a code block:

   ```sql
   SELECT * FROM abyss.lost_entities WHERE id = 'pak_yono';
   ```
2. View the rendered page in a mobile browser or using Chrome DevTools responsive view.
3. Observe how the code block aligns too close to the screen edge.

---

## ✅ Resolution

### 📁 File: `global.css`

The following fixes were applied:

```css
/* Universal fix for <pre> and <code> */
pre, code {
  padding-left: 1rem;
  padding-right: 1rem;
  white-space: pre-wrap;
  word-break: break-word;
  overflow-x: auto;
  border-radius: 0.375rem;
}

/* Tailwind Typography fix */
.prose pre {
  padding-left: 1rem;
  padding-right: 1rem;
  border-radius: 0.375rem;
  white-space: pre-wrap;
  word-break: break-word;
  overflow-x: auto;
  background-color: #0f172a;
  color: #f8fafc;
}

@media (max-width: 640px) {
  pre, code, .prose pre {
    padding-left: 1.25rem;
    padding-right: 1.25rem;
  }
}
```

---

## 📌 Status

* ✅ **Bug confirmed**
* ✅ **Fix implemented**
* ✅ **Tested on multiple screen sizes**
* ⬜ Awaiting pull request review

---

## 🧠 Notes

* This bug is subtle but significantly affects readability on mobile.
* Ensuring proper padding and overflow behavior in code blocks is essential for developer documentation, technical blogs, and MDX-based content rendering.

---

## 👤 Reporter

**Rei Reltroner** – Creator of Reltroner Studio & Astralis System Archivist

---

Let this serve as a reminder: *Clarity in code begins with clarity in UI.*
