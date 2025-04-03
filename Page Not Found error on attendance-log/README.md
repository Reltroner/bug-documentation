# 🐞 Debug Log: `Page Not Found` Error on `/attendance/log`

## 📂 Project Context
This project is an **Employee Attendance Management System** built with:
- **Node.js**, **Express.js**
- **MongoDB (Mongoose)**
- **EJS** for templating
- Role-based access (`Admin`, `Manager`, `Employee`)

---

## ❗ Problem Description

### ❌ Unexpected behavior:
When accessing the route `/attendance/log` from an **Admin** account, the system throws the following error:

```
Error: Page Not Found
    at C:\xampp\htdocs\employee-attendance-system\app.js:111:10
```

### 🔍 Symptoms:
- Sidebar link `/attendance/log` points to the correct path
- Template file `views/attendance/log.ejs` **exists**
- Admin role has proper access
- But it either:
  - Redirects unexpectedly to `/employee`
  - Or hits the global 404 handler

---

## 🔎 Root Cause

### 🧨 1. Duplicate Route Declaration
Inside `routes/attendance.js`:

```js
// ❌ Duplicated route that causes Express to ignore the second one
router.get('/', isAuth, attendance.index);
router.get('/', isAuth, wrapAsync(attendance.roleCheck)); 
```

Express will **only use the first** matching route (`router.get('/')`) and **ignore the rest**, causing misbehavior when `/attendance/log` is accessed.

---

### 🧨 2. Dynamic Route Collisions
Route order matters in Express. This line:

```js
router.get('/:id', ...)
```

Can unintentionally **capture `/log` as a parameter** (i.e., `req.params.id = 'log'`) unless your specific routes (like `/log`) are placed **before** it.

---

## ✅ Solution

### ✅ Step-by-step Fix:

1. **Remove duplicate route definitions** from `routes/attendance.js`.
   ```js
   // ❌ REMOVE THIS if attendance.index is already used
   router.get('/', isAuth, wrapAsync(attendance.roleCheck));
   ```

2. **Ensure route `/log` is placed above any dynamic `/:id` route**:
   ```js
   router.get('/log', isAuth, checkRole(['Admin', 'Manager']), wrapAsync(attendance.viewLog));

   // Place this below:
   router.get('/:id', isAuth, ...);
   ```

3. **Confirm sidebar/admin.ejs or sidebar partial** points correctly to:
   ```ejs
   <a href="/attendance/log" class="list-group-item list-group-item-action bg-light">
     📝 Approval Log
   </a>
   ```

---

## ✅ Result After Fix

- Admin and Manager can now successfully access `/attendance/log`
- View `views/attendance/log.ejs` is rendered correctly
- No more redirection to `/employee` or unexpected 404

---

## 🧠 Lesson Learned

> In Express.js, **route order and uniqueness are critical**.  
> Always define specific routes (e.g., `/log`) **before** dynamic ones (e.g., `/:id`) and **never duplicate** route definitions for the same path.

---

## 🛠 Maintainer Note

If you're modifying routes in `attendance.js`, always validate with:

```bash
# to test routes manually:
curl http://localhost:3000/attendance/log
```

---
Happy debugging, and remember: **logic is the light in the nytherion abyss** ✨  
Let Astralis light the unknown.  
---
