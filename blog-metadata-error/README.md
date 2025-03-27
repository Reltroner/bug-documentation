# Fixing `generateMetadata` and `params` Errors in Next.js Blog Pages

When working with dynamic routes in Next.js App Router (such as `/blog/[slug]`), a few common mistakes can cause your metadata `title` to be incorrect — always showing the default (e.g., `Reltroner Studio`) — or even break the build entirely.

---

## ❌ Common Mistakes

### 1. Awaiting `props` in Server Component
```js
const { params } = await props; // ❌ Wrong
```

**Why it's wrong**: In server components, `props` is not a Promise. Awaiting it will not work and will cause unexpected behavior.

---

### 2. Typo in `generateMetadata` Function Name
```js
export async function geneateMetadata(...) // ❌ Typo
```

**Why it's wrong**: Next.js looks for `generateMetadata` by name. If it's misspelled, metadata won't load, and page `<title>` will always fall back to the layout's default.

---

## ✅ Clean and Correct Solution

### 1. Use Direct Destructuring for `params`
```diff
- export default async function BlogPostPage(props) {
-   const { params } = await props;
+ export default async function BlogPostPage({ params }) {
```

### 2. Correct the Metadata Function Name
```diff
- export async function geneateMetadata({ params: { slug } }) {
+ export async function generateMetadata({ params: { slug } }) {
```

---

## 💡 Explanation

- `props` in server components is **already resolved**, so you don’t need to (and shouldn’t) `await` it.
- The metadata generation function name **must be exact**: `generateMetadata`.
- Titles for dynamic routes like blog posts are correctly injected only when this function is implemented properly.

---

## 🧪 Final Working Code

```js
// ✅ Dynamic Metadata
export async function generateMetadata({ params: { slug } }) {
  const post = await getPost(slug);
  return {
    title: post.title,
    description: post.description,
  };
}

// ✅ Blog Post Page
export default async function BlogPostPage({ params }) {
  const post = await getPost(params.slug);

  if (!post || !post.title) {
    return <div className="p-6 text-red-500 font-bold">Post not found</div>;
  }

  return (
    <div className="max-w-screen-md mx-auto px-4 py-6">
      <Heading>{post.title}</Heading>
      <p className="text-sm text-gray-600">{post.description}</p>
      <p className="italic text-sm text-gray-500 mb-4">
        {post.date} — {post.published ? "Published" : "Draft"}
      </p>
      {post.image && (
        <img
          src={post.image}
          alt={post.title}
          className="w-full h-auto rounded-lg mb-6 object-cover"
        />
      )}
      <article
        className="prose prose-slate max-w-none text-justify"
        dangerouslySetInnerHTML={{ __html: post.html }}
      />
    </div>
  );
}
```

---

## ✅ Result After Fixing

- The page title becomes `Post Title | Reltroner Studio`.
- No more `params.slug` errors or broken metadata.
- Browser tabs and SEO are now accurate and dynamic.

---

Let Astralis light the unknown.

---
