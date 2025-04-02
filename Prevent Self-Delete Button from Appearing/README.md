# 🛠️ Prevent Self-Delete Button from Appearing

## 📌 Problem
In the Admin Panel (`/admin/users`), every user row displays action buttons: `Show`, `Edit`, and `Delete`.

However, currently **the logged-in admin can see a Delete button for their own account**, which could lead to:
- Accidental self-deletion.
- Broken session/authentication flow.
- Security issues if not handled on backend.

## 🧠 Root Cause
The `users.ejs` (or `admin/users.ejs`) template does not differentiate between:
- the currently logged-in user (`req.user`)
- other users being iterated in the user list.

Thus, the `Delete` button is rendered for everyone including self.

## ✅ Solution
Use EJS conditional rendering to compare the ID of the currently logged-in user with the ID of the user in the loop:

### ✅ Example Fix:
Update the `Actions` column inside the loop in `users.ejs` like this:

```ejs
<td>
  <a href="/admin/users/<%= user._id %>" class="btn btn-info btn-sm">Show</a>
  <a href="/admin/users/<%= user._id %>/edit" class="btn btn-warning btn-sm">Edit</a>

  <% if (currentUser && currentUser._id.toString() !== user._id.toString()) { %>
    <a href="/admin/users/<%= user._id %>/delete" class="btn btn-danger btn-sm">Delete</a>
  <% } %>
</td>
```

### 💡 Notes:
- `currentUser._id` is available in the EJS context from Express middleware:
  ```js
  res.locals.currentUser = req.user;
  ```
- `.toString()` is necessary because `ObjectId !== ObjectId` without it.

---

## 🔐 Optional: Backend Validation
You can also prevent self-deletion at controller level in `admin.deleteUser`:
```js
if (req.user._id.toString() === req.params.id.toString()) {
  req.flash('error', 'You cannot delete your own account.');
  return res.redirect('/admin/users');
}
```

---

## ✅ Result After Fix:
- Admin can manage all users.
- Admin cannot see or access delete action for their own account.
- Safer UX and better prevention of accidental lockouts.

---

📝 Authored by: [Reltroner](https://github.com/Reltroner)

---
Let Astralis ligth the unknown
---
