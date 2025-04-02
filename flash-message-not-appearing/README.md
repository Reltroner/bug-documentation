# ⚠️ Flash Message Not Appearing Bug

## 🐞 Bug Description

When submitting a registration form with mismatched `password` and `confirmPassword`, the `req.flash('error_msg', 'Passwords do not match')` message does not appear on the redirected page (`/auth/register` or `/auth/login`).

This issue also occurs for other flash messages such as login success or error, making it hard for users to get feedback.

---

## ✅ Expected Behavior

Flash messages (error or success) should appear correctly on the redirected page after actions like login, registration, or validation errors.

---

## ❌ Actual Behavior

Flash messages are not shown, even though:

- `connect-flash` is already initialized.
- `req.flash(...)` is called in the controller.
- The EJS view includes `partials/alert`.

---

## 🛠 Root Cause

1. **res.locals.messages** was set correctly but other flash messages were consumed earlier:

   ```js
   res.locals.messages = req.flash();         // ✅ Correct
   res.locals.success = req.flash('success'); // ⚠️ Consumes 'success' from flash
   res.locals.error = req.flash('error');     // ⚠️ Consumes 'error' from flash
   ```

   This caused `messages.success` and `messages.error_msg` to be `undefined`.

2. **partials/alert.ejs** was not handling `error_msg` or other flash keys properly.

---

## ✅ Fix Steps

### 1. Update Middleware

In `app.js`, use only `res.locals.messages` to prevent premature consumption of flash data:

```js
app.use((req, res, next) => {
  res.locals.messages = req.flash(); // This includes all flash messages
  res.locals.currentUser = req.user;
  res.locals.user = req.user || null;
  res.locals.title = 'Employee Management'; 
  next();
});
```

---

### 2. Create or Fix `views/layouts/partials/alert.ejs`

```ejs
<% if (messages.error_msg && messages.error_msg.length > 0) { %>
  <div class="alert alert-danger alert-dismissible fade show" role="alert">
    <%= messages.error_msg[0] %>
    <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
  </div>
<% } %>

<% if (messages.success && messages.success.length > 0) { %>
  <div class="alert alert-success alert-dismissible fade show" role="alert">
    <%= messages.success[0] %>
    <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
  </div>
<% } %>

<% if (messages.error && messages.error.length > 0) { %>
  <div class="alert alert-danger alert-dismissible fade show" role="alert">
    <%= messages.error[0] %>
    <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
  </div>
<% } %>
```

---

### 3. Ensure You Include It in Your Views

At the top of your `login.ejs` and `register.ejs`, add:

```ejs
<%- include('../layouts/partials/alert') %>
```

---

## 💡 Optional Improvement

Instead of hardcoding the redirect path in controller:

```js
return res.redirect('/auth/register'); // ❌
```

Use a dynamic referrer if possible:

```js
return res.redirect(req.originalUrl || '/auth/register'); // ✅ fallback
```

---

## 📌 Status

✅ Fixed on commit `xxxxx`  
📁 File: `app.js`, `partials/alert.ejs`, and relevant EJS views.

---

> Let Astralis light the unknown — and flash your errors properly ✨
