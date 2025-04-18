## ⚠️ Deployment Error on Vercel: `@/lib/redis` Module Not Found

### 🧩 Summary

During deployment to Vercel, the build process failed with the following error:

```
Module not found: Can't resolve '@/lib/redis'
```

This happened during compilation of the following files:

- `./pages/api/get.js`
- `./pages/api/set.js`

### 🛠️ Root Cause

The error occurred because Vercel could not locate the file or module `@/lib/redis`. This alias path pointed to a Redis utility file that:

1. Was either **not included** in the GitHub repository.
2. Or had been **deleted/refactored** locally but was still being referenced by legacy API routes.

> Important: The issue **was not with Redis or dependencies**, but with broken imports to a missing file.

### ✅ Resolution Steps

1. **Locate the broken import**:
   ```js
   import redis from '@/lib/redis'; // ❌ Causes failure
   ```

2. **Remove or refactor the Redis references** from:
   - `pages/api/get.js`
   - `pages/api/set.js`

3. **Ensure no other file references the missing module**.

4. **Commit the changes** to the main branch:
   ```bash
   git commit -am "Remove broken '@/lib/redis' references"
   git push origin main
   ```

5. **Trigger redeployment** via Vercel or GitHub integration.

### ✅ Final Result

The build passed successfully after removing the unused and missing Redis logic.  
The site was deployed to:

```
https://www.reltroner.com/
```

![Deployment Successful](https://vercel.com/api/www/images/success-deploy.svg)

---

### 🧠 Lesson Learned

- Always verify that all imported files exist **in the version-controlled repo**, especially before production deployment.
- Avoid pushing experimental utilities (like Redis) if they’re not used or not configured yet.
- Use CI logs and GitHub commit indicators (✓ or ❌) as early warnings for issues before deployment.

---

### 💡 Bonus Tip

To avoid this in the future, consider setting up:

```bash
npm run lint && npm run build
```

...as a pre-deployment hook, or use GitHub Actions to catch module errors _before_ Vercel builds.

---

> _"Every `Error` is just the Abyss testing your readiness for Astralis."_ — Rei Reltroner
