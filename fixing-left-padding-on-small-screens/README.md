---

# ✅ Debug Log: Fixing Left Padding on Small Screens (e.g., iPhone 4)

## 🐞 Issue

When viewed on very small devices like **iPhone 4**, the content of the blog post appeared **too close to the left edge**, especially the `<Heading>`, `description`, and `date`. This caused readability and layout imbalance issues on extra small viewports.

---

## 🔍 Debugging Process

### ✅ Final Fix Applied:

To ensure a consistent **left padding** on all devices including ultra-small screens:

```jsx
return (
  <div className="flex-grow px-4 sm:px-6 md:px-8 py-6 max-w-screen-md mx-auto w-full pl-20">
    <Heading>{post.title}</Heading>
    <p className="text-sm text-gray-600 pl-4">{post.description}</p>
    <p className="italic text-sm text-gray-500 mb-4 pl-4">
      {post.date} — {post.published ? "Published" : "Draft"}
    </p>
    {post.image && (
      <img
        src={post.image}
        alt={post.title}
        className="w-full h-auto rounded-lg mb-6 object-cover sm:px-0 px-4"
      />
    )}
    <article
      className="prose prose-slate max-w-none text-justify sm:px-0 px-4"
      dangerouslySetInnerHTML={{ __html: post.html }}
    />
  </div>
);
```

---

## 💡 Explanation

| Element         | Fix                              | Purpose                                           |
|----------------|-----------------------------------|---------------------------------------------------|
| `div` container| `pl-20`                           | Adds enough left padding to shift everything right on small screens |
| `description` `<p>` | `pl-4`                      | Ensures post description does not align too left  |
| `date` `<p>`   | `pl-4`                            | Same fix for date & published status              |
| `<article>`    | `sm:px-0 px-4`                    | Applies horizontal padding on small screens only  |
| `<img>`        | `sm:px-0 px-4`                    | Keeps image aligned with text                     |

---

## ✅ Result After Fix

- 🎯 Title, description, and metadata now have **breathing room** from the left edge.
- 📱 Looks clean and readable even on **iPhone 4 width (320px)**.
- 🧩 Layout consistency across devices is **preserved**.
- 🧠 Readability and visual polish improved significantly.

---

## 📌 Notes

If you're using a global layout, it’s better to **abstract this behavior** using responsive `padding-left` utilities (`pl-4`, `pl-6`, etc.) and **avoid hardcoding pixel shifts unless necessary**.

---

Let Astralis light the unknown ✨  
— Documented by Reltroner Studio
