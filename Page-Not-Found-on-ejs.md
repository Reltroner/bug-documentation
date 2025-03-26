# Error Documentation: Page Not Found on /admin/users Search

## Title
**Error**: Page Not Found when clicking the Search button on `/admin/users`

## Symptoms
When the "Search" button is clicked on the `/admin/users` page, the application throws:

```
Error: Page Not Found at app.js:81:10
```

## File Analysis

### 1. `admin/users.ejs`
- The search form action must point to `/admin/users`.
- **Issue**: Previously pointed to `/admin/index` or `/users`.
- **Fix**: Updated to `action="/admin/users"`

### 2. `routes/admin.js`
- Route `GET /admin/users` is correctly defined and calls `admin.index`.
- Protected using `isAuth` and `checkRole(['Admin'])`.

### 3. `controllers/admin.js`
- `admin.index()` properly handles the search query.
- Renders `admin/users.ejs` with `title`, `users`, and `searchQuery`.

### 4. `app.js`
- `/admin` route is registered using `app.use('/admin', adminRoutes)`.
- 404 handler is correctly defined using `app.all('*', ...)`.

### 5. `sidebar.ejs` / `navbar.ejs`
- Ensure all links reference `/admin/users` instead of `/users`.

## Root Cause
- The search form and links pointed to a non-existent `/users` route instead of `/admin/users`.
- Although routes and controllers were defined properly, the path mismatch caused a 404 error.

## Solution Applied
- Fixed the search form action to `/admin/users`.
- Updated all action links in `admin/users.ejs` to `/admin/users/:id`.
- Verified correct routing and controller integration.

## Additional Recommendations
- Review and fix all `sidebar` / `navbar` links pointing to `/users`.
- Sanitize and validate the `searchQuery` server-side.
- Optionally include a "Reset" button to clear the search input.

## Final Status
After correcting the form and link paths, the search functionality on `/admin/users` works without error.
