## ⚠️ Common Error: MissingSchemaError for User

### ❓ Problem
When running `node seeds/attendance.js`, the following error occurs:

```
MissingSchemaError: Schema hasn't been registered for model "User".
Use mongoose.model(name, schema)
```

### 🔍 Cause
This error occurs because Mongoose is trying to resolve a schema reference (using `ref: 'User'`) but the `User` model has not been imported or registered in the current file.

In this project:
- The `Attendance` model references the `Employee` model
- The `Employee` model references the `User` model via:
```js
user: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true }
```

If `User` is not imported in the seeder or any file that triggers population of this chain, Mongoose throws `MissingSchemaError`.

### ✅ Solution
To fix this error, **import the `User` model** in `seeds/attendance.js`:

```js
const User = require('../models/user'); // ✅ Required for Mongoose to register the schema
```

Even if you're not directly using the `User` model in that file, you need it for the population chain to work correctly.

### 🧠 Best Practice
Ensure that **all referenced models (via `ref`) are registered** using `require()` before calling `.populate()` or any related operation in your seeders or logic.

This avoids hidden runtime errors that can halt the seeding process.

---
This is especially important when using deeply nested references like:
```
Attendance ➝ Employee ➝ User
```
Where `Attendance` references `Employee`, and `Employee` references `User`. All three schemas must be known to Mongoose.

✅ Fix this once, and future seeding will work smoothly!

###Let Astralis Light the Unknown ✨
---
