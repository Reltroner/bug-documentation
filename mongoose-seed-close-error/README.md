# ❌ Mongoose Seed Script Error: `Connection.prototype.close() no longer accepts a callback`

**Location**: `seeds/user.js`  
**Author**: Reltroner  
**Category**: MongoDB / Mongoose  
**Error Type**: Breaking Change (Mongoose v7+)

---

## 💥 Error Overview

When running a seed script (`node seeds/user.js`) to populate dummy users into MongoDB, the following error appears:

```bash
MongooseError: Connection.prototype.close() no longer accepts a callback
```

Stack trace:
```
.../node_modules/mongoose/lib/connection.js:1194
throw new MongooseError('Connection.prototype.close() no longer accepts a callback');
```

---

## 🧠 Root Cause

In Mongoose **v7 and above**, the `.close()` method no longer accepts a callback function.

### ❌ Old Usage (no longer valid):
```js
mongoose.connection.close(() => {
  console.log('Connection closed');
});
```

---

## ✅ Correct Usage (Mongoose v7+)

### ✅ Use `await` because `.close()` now returns a Promise:
```js
await mongoose.connection.close();
console.log('🌱 Seeding complete & MongoDB connection closed');
```

If you're not inside an async function, wrap the logic in one or use `.then()` like this:

```js
mongoose.connection.close()
  .then(() => console.log('Connection closed'))
  .catch(err => console.error('Close failed:', err));
```

---

## ✅ Recommended Seed Script Structure

```js
const mongoose = require('mongoose');
const User = require('../models/user');

mongoose.connect('mongodb://127.0.0.1/employee-attendance')
  .then(() => console.log('✅ MongoDB Connected'))
  .catch(err => console.error('❌ Connection Error:', err));

const seedUsers = async () => {
  try {
    await User.deleteMany({}); // Clear existing data

    // ... your dummy user creation logic here ...

    await mongoose.connection.close();
    console.log('✅ Seeding complete & connection closed');
  } catch (err) {
    console.error('❌ Seeding failed:', err);
    await mongoose.connection.close();
  }
};

seedUsers();
```

---

## 🔒 Additional Notes

You might also see this MongoDB shell warning:

```bash
Access control is not enabled for the database. Read and write access to data and configuration is unrestricted.
```

This is **not an error**, just a **security warning** for local development. You can ignore it unless you're deploying to production.

---

## 🔁 TL;DR

| ❌ Before                     | ✅ After                            |
|-----------------------------|-------------------------------------|
| `mongoose.connection.close(cb)` | `await mongoose.connection.close()`   |

---

### ✨ Written by Reltroner  
*“Understand your errors. Respect your stack trace. Master your flow.”*

Let Astralis light the unknown 🔴
