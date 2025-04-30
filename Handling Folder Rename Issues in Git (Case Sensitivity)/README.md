# 🧠 Handling Folder Rename Issues in Git (Case Sensitivity)

In cross-platform development, especially when switching between Windows, macOS, and Linux, developers often face an issue where renaming a folder by only changing the capitalization (e.g., `Items` → `items`) is **not detected by Git**.

This can lead to problems where the **code appears renamed locally**, but GitHub or other collaborators **do not see any change**, causing confusion or build errors.

---

## ❗ Problem Statement

> Renaming a folder (e.g., `Items` → `items`) in VS Code or a file manager does **not update the folder name in the Git repository**, especially on Windows/macOS, because:

- Git is **case-sensitive**, but...
- Windows and macOS file systems are **case-insensitive by default**.

This results in **Git ignoring case-only renames**.

---

## ✅ Solution: Rename in Two Steps

To ensure Git recognizes the folder rename, use a two-step rename process:

### Option 1 – Rename with Temporary Name
```bash
# Step 1: Rename to a temporary name
git mv Items temp_folder

git commit -m "Rename Items to temp_folder"

# Step 2: Rename to the desired lowercase name
git mv temp_folder items

git commit -m "Rename temp_folder to items"
```

### Option 2 – Rename Manually via File System
1. Rename `Items/` → `items_temp/`
2. Commit the change:
   ```bash
   git add -A
   git commit -m "Rename Items to items_temp"
   ```
3. Rename `items_temp/` → `items/`
4. Commit again:
   ```bash
   git add -A
   git commit -m "Rename items_temp to items"
   ```

---

## 🔍 Why Git Ignores It

Git tracks filenames and their cases internally. But on **case-insensitive** systems like Windows/macOS:
- Renaming `Items` → `items` happens at the OS level but looks the same to Git.
- Git doesn’t recognize it as a "real change" unless you **force the rename** via intermediate names.

---

## 🧼 Additional Tips

- If the folder is empty, Git won’t track it at all.
  > Add a placeholder file like `.gitkeep` inside the folder.

- Run `git status` to verify changes.

- Always test the renamed folder in your codebase to ensure imports and paths are resolved correctly.

---

## 📝 Summary

| Issue                     | Cause                                | Solution                       |
|---------------------------|--------------------------------------|--------------------------------|
| Folder rename not detected| Case-insensitive file system         | Rename in two steps            |
| Folder not tracked        | Git ignores empty folders            | Add `.gitkeep` or real files   |

---

## ✨ Author
**Reltroner Studio** – Worldbuilding-driven development for Asthortera & beyond 🌌

> Let Astralis light the unknown.
