# ⚔️ Git Case Rename War: Surviving the `Item` → `item` Curse

> _"It looked fine on localhost. But when I pushed to production... everything shattered."_

This is the true story of a battle every developer must face at some point: the **case-sensitive rename war** in Git. This time, the enemy was subtle:

```
app/Item → app/item
```

## 🧨 The Illusion Trap

### ✅ What happens locally (on Windows/macOS):
- You rename the folder from `Item` to `item`.
- `npm run dev` works fine.
- Everything seems okay.

### ❌ What happens on GitHub / production (Linux):
- Git does NOT detect the rename — **because the file system is case-insensitive**.
- GitHub still sees the old `Item/` folder.
- Your deployed app throws **404 errors**, broken imports, and chaos.

> **The folder was renamed in your mind, but not in the repo.**

## 🧠 Why It Happens
- Windows & macOS file systems are **case-insensitive**, so `Item` = `item`.
- Git internally tracks **case-sensitive** changes.
- But **Git will NOT stage a case-only rename unless forced**, leading to ghost folders and broken references.

## 💀 The Fallout
- `/items` route returns 404
- Next.js cannot resolve `page.jsx` in the renamed directory
- Imports fail silently
- Markdown links break
- You start doubting your sanity

> _"I committed. I pushed. Why is the server acting like it's 2 commits behind!?"_

---

## 🛠️ The Fix: The Two-Step Rename Ritual

To properly rename a folder by changing only its capitalization, use this ritual:

```bash
# Step 1: Rename to a temporary folder
git mv Item temp_folder

git commit -m "Rename Item to temp_folder"

# Step 2: Rename to lowercase
git mv temp_folder item

git commit -m "Rename temp_folder to item"

git push
```

Now Git sees **real changes**, your repo is clean, and the server won't cry anymore.

---

## 🧬 The Deeper Lesson

This is more than a tech bug. It’s a reminder:
- **Not everything that works locally will survive production.**
- **Perception is not reality.**
- **Even a single letter — one capital `I` — can cause collapse.**

> _"Naming is hard. Case renames are harder. Surviving Git’s indifference to your pain? That’s elite."_

---

## 📦 Bonus Tip
If Git still doesn’t detect changes:
```bash
git config core.ignorecase false
```
Then redo the rename. This tells Git to be case-sensitive even on Windows/macOS.

---

## 📝 Summary

| Problem                      | Cause                         | Fix                            |
|-----------------------------|-------------------------------|---------------------------------|
| Folder rename not detected  | Case-only rename on insensitive FS | Two-step rename (`temp` first) |
| Server throws 404           | Old folder not removed        | Git didn't track rename        |
| Markdown/imports break      | Misaligned casing             | Commit case-correct folder     |

---

## 🔥 Final Words
> _"The real bug wasn’t in the code. It was in trusting the file system too much."_

— **Reltroner**, post-war survivor, commit `c77077c`

🌀 Return to peace: `git push origin main && breathe`

---

> Let Astralis light the unknown — even in your Git history.

---

📄 Read the Handling Folder Rename Issues in Git (Case Sensitivity)
: [Handling Folder Rename Issues in Git (Case Sensitivity)
](./README.md)
