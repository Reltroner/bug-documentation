# ⚠️ Bug Documentation (by Reltroner)

This repository is a growing collection of error handling logs, solutions, and personal notes on real-world issues encountered in web development — including Laravel, Express.js, EJS, MongoDB, Tailwind, Blade, Bootstrap, Flatpickr, and more.

Every file in this repo is a direct battle log. I don't just fix — I document and share.

## 🔍 Topics Covered

- 🧭 `Page Not Found` on dynamic routes (EJS / Express / Next.js)
- 🧱 MongoDB query and schema validation errors
- 🎯 Bootstrap + Tailwind layout and script conflicts
- 🔀 Middleware errors & async callback bugs
- 🧪 `params.slug` & layout bugs in Next.js App Router
- 🛠️ Flatpickr + `datetime` format incompatibility
- ⚙️ Laravel migration rollback & model cast mismatch
- 🔄 Folder renaming issues in Git (case-sensitive FS)
- 🕛 `check_in` and `check_out` showing `00:00` due to wrong column type
- ⚡ Deployment issues on Vercel / Netlify (`Module Not Found`, `esbuild`, etc.)
- 🔐 Preventing self-delete in user management
- 🔍 Google Analytics: filter internal traffic with GTM

## 📌 Recent Additions

- ✅ `Presence check_in and check_out Column Type`: Fixed incorrect schema type from `date` to `datetime` that caused time values to be lost in Laravel HR app.
- 🛡️ `Prevent Self-Delete`: Fix logic so that users can’t delete themselves.
- ⚠️ `MissingSchemaError for User`: Documented a mongoose schema import failure.
- 🧭 `attendance-log` 404 error: Wrong route/method mismatch.
- 📊 `Flatpickr datetime + model cast`: Synced front-end picker with Laravel model casting.

## 🚀 Why This Exists

Because learning from errors is not weakness.  
It’s the foundation of Astralis Pinnacle.

> Let Astralis light the unknown.
