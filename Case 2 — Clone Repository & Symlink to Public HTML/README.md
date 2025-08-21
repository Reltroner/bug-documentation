# 🛠️ Case 2 — Clone Repository & Symlink to Public HTML

This document describes the process of cloning the Laravel project repository from GitHub and linking its `public` directory to the server’s `public_html` for web access.

---

## 1. Create Repositories Directory and Clone Project

Navigate to the home directory and clone the repository:

```bash
mkdir -p ~/repositories && cd ~/repositories
git clone git@github.com:Reltroner/reltroner-hr-app.git
````

**Result:**

* Repository cloned into: `~/repositories/reltroner-hr-app`

---

## 2. Remove Existing `public_html`

Since cPanel uses `~/public_html` as the default web root, remove any existing directory or symlink:

```bash
rm -rf ~/public_html
```

---

## 3. Create Symlink to Laravel’s Public Directory

Link the Laravel `public` folder to the default `public_html`:

```bash
ln -s ~/repositories/reltroner-hr-app/public ~/public_html
```

**Explanation:**

* `~/public_html` → now points to `~/repositories/reltroner-hr-app/public`
* Ensures the server serves the Laravel application correctly.

---

## 4. Verification

Check if the symlink was created successfully:

```bash
ls -l ~ | grep public_html
```

**Expected Output:**

```
public_html -> /home/ynoqpang/repositories/reltroner-hr-app/public
```

---

## ✅ Outcome

* Repository successfully cloned from GitHub.
* Default `public_html` replaced with a symlink to Laravel’s `public` directory.
* Web server now serves the Laravel app directly from the cloned repository.

