# 🐛 Bug Report — 404 Not Found Due to Incorrect Document Root

This document records the issue where all probe test files returned **404 Not Found**, caused by the domain being pointed to the wrong document root.

---

## 1. Problem Description

Probes were created in different locations:

```bash
echo "FROM_PUBLIC" > /home/ynoqpang/repositories/reltroner-hr-app/public/which.txt
echo "FROM_SUBROOT" > /home/ynoqpang/hrm.reltroner.com/which.txt
echo "FROM_PUBLIC" >/home/ynoqpang/repositories/reltroner-hr-app/public/probe_public.txt
echo "FROM_SUBROOT" >/home/ynoqpang/hrm.reltroner.com/probe_subroot.txt
echo "FROM_PUBLIC_HTML" >/home/ynoqpang/public_html/probe_main.txt
````

Accessing them returned 404:

* [https://hrm.reltroner.com/probe\_public.txt](https://hrm.reltroner.com/probe_public.txt) → **404**
* [https://hrm.reltroner.com/probe\_subroot.txt](https://hrm.reltroner.com/probe_subroot.txt) → **404**
* [https://hrm.reltroner.com/probe\_main.txt](https://hrm.reltroner.com/probe_main.txt) → **404**

---

## 2. Root Cause

* All probes failed, meaning **LiteSpeed is not pointing to the intended document root**.
* In cPanel, the configured **Document Root** for `hrm.reltroner.com` was incorrect and showed duplication:

```
/home/ynoqpang/home/ynoqpang/repositories/reltroner-hr-app/public
```

* Additionally, LiteSpeed commonly **blocks symlinked docroots**, so using `~/hrm.reltroner.com -> public` will not work.

---

## 3. Solution

### Step 1 — Fix Document Root

In **cPanel → Domains → Manage** for `hrm.reltroner.com`:

* Set **Document Root** to:

```
/home/ynoqpang/repositories/reltroner-hr-app/public
```

⚠️ Do **not** use symlinks for document root.

---

### Step 2 — Remove Conflicting Folder

Delete the empty subroot folder to avoid confusion:

```bash
rm -rf /home/ynoqpang/hrm.reltroner.com
```

---

### Step 3 — Verify with Probe

Create a fresh probe:

```bash
echo FROM_PUBLIC > /home/ynoqpang/repositories/reltroner-hr-app/public/probe_check.txt
```

Check in browser:

```
https://hrm.reltroner.com/probe_check.txt
```

✅ Expected output:

```
FROM_PUBLIC
```

---

## 4. Expected Outcome

* With the correct document root, probe files under `public/` are served successfully.
* Laravel’s `index.php` in the `public` folder loads correctly, instead of `404` or `Index of /`.

---

## ✅ Resolution Status

* Cause: Misconfigured document root (`/home/ynoqpang/home/ynoqpang/...`) and symlink restrictions.
* Fix: Point domain directly to `/home/ynoqpang/repositories/reltroner-hr-app/public` in cPanel.
* Result: 404 errors resolved, app loads properly.

