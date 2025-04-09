# 📚 Blog Search – Reltroner Studio

A simple yet effective **local search implementation** using `useState` and `useMemo` in **Next.js App Router**.  
This feature is part of the `reltroner.com` project, enabling users to **search blog articles dynamically** without needing global indexing tools.

---

## ✨ Features

- 🔍 Real-time search filtering
- 🧠 Case-insensitive search across:
  - Blog title
  - Blog description
  - Slug (URL)
- 💡 Powered by `useMemo` for performance
- 💅 Smooth UI with TailwindCSS utility classes
- 🎯 Search is scoped **only to the `/blog` route** (not site-wide)

---

## 🧠 Tech Stack

- **Next.js App Router**
- **React Hooks** (`useState`, `useMemo`)
- **TailwindCSS**
- No third-party dependencies required

---

## 🏗️ File Structure

```bash
app/
├── blog/
│   ├── BlogClient.jsx      # Client-side component with search logic
│   └── page.jsx            # Server component with metadata + posts
```

---

## 💻 How It Works

### `page.jsx` (Server Component)

```jsx
import BlogClient from "./BlogClient";

export const metadata = {
  title: "Blog",
  description: "Explore recent articles about world-building and events.",
};

const posts = [/* array of blog objects */];

export default function Blog() {
  return <BlogClient posts={posts} />;
}
```

---

### `BlogClient.jsx` (Client Component)

```jsx
'use client';
import { useState, useMemo } from "react";

export default function BlogClient({ posts }) {
  const [searchQuery, setSearchQuery] = useState("");

  const filteredPosts = useMemo(() => {
    const q = searchQuery.toLowerCase();
    return posts.filter(
      (post) =>
        post.title.toLowerCase().includes(q) ||
        post.description.toLowerCase().includes(q) ||
        post.slug.toLowerCase().includes(q)
    );
  }, [searchQuery, posts]);

  // Render search input and filtered cards...
}
```

---

## 🖼️ Preview

![Search Box in Action](./public/images/search-preview.png)

---

## 📌 Notes

- This is not a full-site search (e.g., Algolia). It’s **scoped to the blog** page only.
- Perfect for small to medium blogs using hardcoded or statically imported content.
- You can easily adapt this to use `Contentlayer`, `MDX`, or CMS-based data sources.

---

## 🧪 Future Improvements

- Add fuzzy search or highlight keywords
- Add filter by tag, author, or date
- Integrate MD content or markdown metadata using `contentlayer`

---

## ☀️ Credits

Crafted with discipline, philosophy, and clarity by **Rei Reltroner**.  
A tribute to the spirit of **Astralis Pinnacle** — conscious creation over chaos.

---

## 🔗 Live Demo

Check out the working version on:  
🌐 **[reltroner.com/blog](https://www.reltroner.com/blog)**

---

> Let Astralis light the unknown.
