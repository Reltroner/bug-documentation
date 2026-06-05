# Debugging Report: YAML Validation, Content Frontmatter, and Next.js Development Server Recovery

**Project:** Reltroner Studio
**Repository:** reltroner-studio
**Date:** 2026-06-06
**Framework:** Next.js 16.2.6 (Turbopack)
**Author:** Rei Reltroner

---

# Executive Summary

This debugging session focused on resolving content validation failures, YAML frontmatter parsing issues, and a Next.js development server startup conflict.

The project initially experienced:

* YAML parsing failures
* Content validation errors
* Hundreds of metadata warnings
* Development server startup conflicts
* Uncertainty regarding frontmatter formatting consistency

At the end of the debugging process:

* All blocking YAML errors were resolved.
* Content validation completed with zero errors.
* Production build completed successfully.
* Static site generation completed successfully.
* Sitemap generation completed successfully.
* Development server conflict was resolved.
* Remaining issues are non-blocking metadata warnings.

---

# Initial Symptoms

The project initially reported:

```bash
YAMLException
```

during development execution.

Content validation also reported:

```bash
[content:validate] Completed with 1 error(s)
```

alongside hundreds of warnings.

Examples included:

```text
missing category
missing title
missing description
invalid tags
```

---

# Investigation Phase 1 — Frontmatter Validation

A representative file was inspected:

```yaml
---
title: "Kalgered–Stelpadland Strategic Buffer Civilization Doctrine"
description: "Official geopolitical doctrine..."
image: "/images/kalgered-stelpadland-doctrine.webp"
author: "Rei Reltroner"
date: "2026-06-05"
published: true
category: "Geopolitics"
tags:
  - Kalgered
  - Stelpadland
  - Asthortera
---
```

The structure appeared valid.

---

# Investigation Phase 2 — Hidden Character Analysis

Character-level inspection was performed:

```powershell
$content.Substring(
    $content.IndexOf('tags:'),
    120
).ToCharArray()
```

Output revealed:

```text
tags:
 U+000D
 U+000A
 TAB
-
```

Analysis confirmed:

* YAML list structure was valid.
* Line endings were valid CRLF.
* No BOM corruption detected.
* No hidden Unicode characters causing parser failures.

---

# Investigation Phase 3 — Content Validation

Validation was executed:

```bash
npm run content:validate
```

Initial result:

```text
Completed with 1 error(s)
```

After corrections:

```text
[content:validate] Completed with 0 error(s)
```

This confirmed that all blocking content validation errors had been eliminated.

---

# Investigation Phase 4 — Production Build Verification

Production build was executed:

```bash
npm run build
```

Result:

```text
✓ Compiled successfully
✓ Finished TypeScript
✓ Collecting page data
✓ Generating static pages
✓ Finalizing page optimization
✓ next-sitemap
```

Generated routes included:

* Blog pages
* Events pages
* Places pages
* Characters pages
* Factions pages
* Laws pages
* Philosophies pages
* Technologies pages

A total of hundreds of static pages were successfully generated.

---

# Investigation Phase 5 — Development Server Failure

When starting development mode:

```bash
npm run dev
```

Next.js reported:

```text
Another next dev server is already running.

Local: http://localhost:3006
PID: 16464
```

---

# Investigation Phase 6 — Process Verification

PID verification:

```powershell
Get-Process -Id 16464
```

Result:

```text
Cannot find a process with the process identifier 16464
```

Additional verification:

```powershell
taskkill /PID 16464 /F
```

Result:

```text
The process was not found.
```

This indicated that the reported PID no longer existed.

---

# Investigation Phase 7 — Port Analysis

Port analysis:

```powershell
netstat -ano | findstr :3006
```

Result:

```text
(no output)
```

Meaning:

* Port 3006 was not occupied.
* No active process was listening.

---

# Investigation Phase 8 — Node Process Analysis

Verification:

```powershell
Get-Process node
```

Result:

```text
(no output)
```

Meaning:

* No Node.js process was active.
* No orphaned Next.js runtime remained.

---

# Investigation Phase 9 — Cache and State Cleanup

The Next.js cache directory was removed:

```powershell
Remove-Item .next -Recurse -Force
```

Verification:

```powershell
Test-Path .next
```

Result:

```text
False
```

This confirmed complete cache removal.

---

# Investigation Phase 10 — Development Server Recovery

After cache removal:

```bash
npm run dev
```

Result:

```text
▲ Next.js 16.2.6 (Turbopack)

✓ Ready in 636ms
```

Development mode successfully recovered.

---

# Hydration Warning Investigation

After recovery, the following warning appeared:

```text
A tree hydrated but some attributes of the server rendered HTML didn't match the client properties.
```

React highlighted:

```html
data-new-gr-c-s-check-loaded="14.1299.0"
data-gr-ext-installed=""
```

These attributes are commonly injected by browser extensions such as:

* Grammarly
* Grammarly Beta
* Grammar Checker extensions
* AI writing assistant extensions

The warning does not indicate an application defect.

The warning is generated because browser extensions modify the DOM before React hydration.

---

# Final Status

## Build Status

| Component              | Status  |
| ---------------------- | ------- |
| YAML Parsing           | SUCCESS |
| Frontmatter Validation | SUCCESS |
| Content Validation     | SUCCESS |
| Production Build       | SUCCESS |
| Static Generation      | SUCCESS |
| TypeScript Compilation | SUCCESS |
| Sitemap Generation     | SUCCESS |
| Development Server     | SUCCESS |

---

## Remaining Warnings

The project still contains metadata warnings such as:

```text
missing category
missing title
missing description
invalid tags
```

These warnings:

* Do not block builds.
* Do not break runtime execution.
* Do not prevent deployment.
* Do not prevent static generation.

They are content quality improvements rather than system failures.

---

# Root Cause Summary

The original blocking issue was not caused by:

* Next.js
* Turbopack
* TypeScript
* React
* Static generation

The issue originated from content validation and YAML/frontmatter inconsistencies within the content repository.

After validation corrections and cache cleanup:

* Validation errors reached zero.
* Production build completed successfully.
* Development environment recovered fully.

---

# Conclusion

The debugging objective was successfully completed.

Final project state:

```text
Content Validation Errors: 0
Production Build: PASS
Static Generation: PASS
Sitemap Generation: PASS
Development Server: PASS
```

The Reltroner Studio content system and Next.js application are now operational and ready for continued content development and deployment.
