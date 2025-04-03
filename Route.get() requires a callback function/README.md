## ⚠️ Common Error: Route.get() requires a callback function

### ❓ Problem
When starting the server or logging in as a Manager, the following error occurs:

```
Error: Route.get() requires a callback function but got a [object Undefined]
    at Route.<computed> [as get] (.../node_modules/express/lib/router/route.js:216:15)
```

### 🔍 Cause
This error typically happens when a route in `routes/dashboard.js` is referencing a controller function that is not defined or not exported properly.

In this case, the route:
```js
router.get('/manager', isAuth, checkRole(['Manager']), dashboard.managerDashboard);
```
...fails because `managerDashboard` was **undefined** in `controllers/dashboard.js`.

### ✅ Solution
Make sure the function `managerDashboard` is correctly defined and exported in `controllers/dashboard.js`, for example:

```js
module.exports.managerDashboard = async (req, res) => {
  try {
    const user = await User.findById(req.user._id);
    const employees = await Employee.find({ managerId: user._id }).populate('user');

    const today = new Date();
    today.setHours(0, 0, 0, 0);

    const teamAttendance = employees.map(emp => {
      const todayRecord = emp.attendance.find(att => new Date(att.date).toDateString() === today.toDateString());
      return { employee: emp, attendance: todayRecord };
    });

    res.render('dashboard/manager', {
      user,
      teamAttendance
    });
  } catch (err) {
    console.error(err);
    req.flash('error_msg', 'Failed to load manager dashboard');
    res.redirect('/');
  }
};
```

Then restart your server:
```bash
nodemon app.js
```

### 🧠 Tips
- Always confirm your controller methods exist before calling them in routes
- Use console logs or IDE autocomplete to ensure no `undefined` is being referenced

---
This documentation helps future developers avoid the same mistake when modifying dashboard logic.
