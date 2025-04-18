# ⚠️ Fixing `params.slug` Error in Next.js 14+ App Router

## ❗ Problem

When using dynamic routes with `generateMetadata()` or page components in Next.js App Router, you might encounter the following error:

```
Error: Route "/organizations/[slug]" used `params.slug`. 
`params` should be awaited before using its properties.
Learn more: https://nextjs.org/docs/messages/sync-dynamic-apis
```

This occurs in both:

- `generateMetadata({ params })`
- The actual page component function such as `OrganizationPage({ params })`

---

## 🔍 Root Cause

Starting in Next.js 14+ App Router, the `params` object passed into **server components** and **metadata functions** may be wrapped in a `Promise`.

This means you **cannot directly access** `params.slug` synchronously.  
You must **await** `params` before using any properties.

Also:  
If you're using `.jsx` files instead of `.tsx`, **TypeScript annotations will throw syntax errors**. JSX does not support inline type declarations like:

```js
{ params }: { params: Promise<{ slug: string }> }
```

---

## ✅ Fix (JavaScript/JSX Version)

Update your dynamic route like this:

```js
// app/organizations/[slug]/page.jsx
export default async function OrganizationPage({ params }) {
  const { slug } = await params;
  const { data, contentHtml } = await getOrganization(slug);
  // ...
}
```

And for metadata:

```js
// app/organizations/[slug]/page.jsx
export async function generateMetadata({ params }) {
  const { slug } = await params;
  const { data } = await getOrganization(slug);
  return {
    title: data.title,
    description: data.description,
  };
}
```

---

## ✅ Fix (TypeScript/TSX Version)

If you are using TypeScript, change your file extension to `.tsx` and annotate properly:

```ts
// app/organizations/[slug]/page.tsx
export async function generateMetadata(
  { params }: { params: Promise<{ slug: string }> }
) {
  const { slug } = await params;
  const { data } = await getOrganization(slug);
  return {
    title: data.title,
    description: data.description,
  };
}

export default async function OrganizationPage(
  { params }: { params: Promise<{ slug: string }> }
) {
  const { slug } = await params;
  const { data, contentHtml } = await getOrganization(slug);
  // ...
}
```

---

## 🧠 Tips

- Never write TypeScript-style annotations in `.jsx` files.
- Await `params` first before destructuring any of its fields.
- Check the official Next.js docs on metadata:  
  https://nextjs.org/docs/app/api-reference/functions/generate-metadata

---

## 🏷️ Related Keywords

`generateMetadata`, `params.slug`, `dynamic routes`, `Next.js App Router`, `Promise`, `JSX vs TSX`, `page.jsx`, `slug`, `metadata error`

---

## ✨ Maintained by Reltroner Studio

Documented as part of the Reltroner Worldbuilding Framework.  
Visit: [https://reltroner.com](https://reltroner.com)
