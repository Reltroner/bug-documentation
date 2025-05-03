# 🐛 React Client-Side Exception: Invalid Date Object in JSX

## ❗ Issue Summary

While deploying a dynamic page in a Next.js application (`/laws/[slug]` route), the following error occurred on the client:

```
Application error: a client-side exception has occurred while loading www.reltroner.com (see the browser console for more information).

Error: Objects are not valid as a React child (found: [object Date]). If you meant to render a collection of children, use an array instead.
```

## 🧠 Root Cause

React does **not allow JavaScript `Date` objects** to be rendered directly in JSX. Attempting to render a raw `Date` object like:

```jsx
<p>{{data.date}}</p>
```

...will throw this error because `data.date` is an object, not a string.

## ✅ Solution

Convert the `Date` object to a string format that React can safely render. For example:

```jsx
<p>{{new Date(data.date).toLocaleDateString()}}</p>
```

Alternatively, you can use `toISOString()` for a consistent output:

```jsx
<p>{{new Date(data.date).toISOString()}}</p>
```

## 🔧 Fix Applied

In the `LawPage` component:

```diff
- <li className="italic text-sm pb-2">{{data.date}} • {{data.published ? 'Published' : 'Draft'}}</li>
+ <li className="italic text-sm pb-2">{{new Date(data.date).toLocaleDateString()}} • {{data.published ? 'Published' : 'Draft'}}</li>
```

This ensures that the value passed to JSX is a string, preventing the exception.

## 📁 Affected Files

- `app/laws/[slug]/page.jsx`

## 🔍 Recommendation

Audit all pages that render `data.date`, `data.modified`, or any `Date` object and ensure proper formatting is applied before rendering them in JSX.

---

**Let Astralis light the unknown.**
