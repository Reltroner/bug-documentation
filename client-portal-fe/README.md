# Client Portal Frontend Debugging Documentation

## Project

**Project name:** `client-portal-fe`
**Production domain:** `https://client.skill-wanderer.com`
**Identity provider:** Keycloak
**Realm:** `client-portal`
**OIDC client:** `client-portal-fe`
**Deployment target:** Cloudflare Workers with OpenNext
**Framework:** Next.js
**Runtime platform:** Cloudflare Workers / Wrangler / OpenNext Cloudflare

---

# 1. Executive Summary

This document records the full debugging process performed for the `client-portal-fe` authentication and deployment flow.

The debugging process uncovered several independent issues across different layers:

1. DNS and custom domain configuration needed to be separated from application build problems.
2. Cloudflare deployment was initially overwriting remote Worker environment variables.
3. Runtime OIDC configuration was missing or invalid after deployment.
4. Wrangler required Node.js 22, while the build environment had been temporarily pinned to Node.js 20.
5. The login page entered an authentication bootstrap loop.
6. Keycloak token endpoint returned HTTP 400 during silent refresh.
7. React threw hydration error `#418` because the server-rendered HTML and browser-rendered auth state could diverge.
8. The frontend was treating stale OIDC browser sessions as fatal bootstrap failures instead of unauthenticated state.
9. `automaticSilentRenew`, `monitorSession`, and manual `startSilentRenew()` caused repeated silent-renew attempts against stale refresh tokens.

The final root cause was not a single Keycloak setting. It was a frontend auth-state handling issue: stale OIDC browser storage triggered silent token renewal, Keycloak rejected the refresh attempt, and the frontend repeatedly retried the same flow while rendering different UI states.

---

# 2. Initial Situation

The original deployment and access problem started with the frontend site not being reachable through the intended production domain.

The browser showed:

```txt
DNS_PROBE_FINISHED_NXDOMAIN
```

This indicated that the browser could not resolve the hostname at the DNS layer.

At the same time, the Cloudflare deployment logs showed that the Next.js build and asset upload were mostly successful. Therefore, the first conclusion was:

```txt
The application build was not the primary cause of NXDOMAIN.
The DNS authority and DNS records had to be corrected first.
```

---

# 3. DNS Debugging

## 3.1 Observed DNS Setup

The domain `reltroner.com` was still using Hostinger nameservers:

```txt
ns1.dns-parking.com
ns2.dns-parking.com
```

At the same time, records were also configured inside Cloudflare DNS.

This created a split-brain expectation:

```txt
Hostinger was authoritative for the domain,
but some records were being edited inside Cloudflare DNS.
```

Because the authoritative nameservers were Hostinger nameservers, public DNS resolvers would read the Hostinger DNS zone, not the Cloudflare DNS zone.

## 3.2 DNS Decision

The correct architecture was:

```txt
reltroner.com authoritative DNS:
Hostinger nameservers

client / lms / subdomains:
CNAME records created inside Hostinger DNS zone
```

For `lms.reltroner.com`, the correct pattern was:

```txt
Type: CNAME
Name: lms
Target: <cloudflare-pages-project>.pages.dev
```

For `client.skill-wanderer.com`, the same principle applied depending on which nameserver was authoritative for `skill-wanderer.com`.

## 3.3 DNS Lesson

The important distinction:

```txt
Cloudflare dashboard DNS records are only authoritative
if the domain uses Cloudflare nameservers.
```

If the domain uses Hostinger nameservers, DNS records must be configured in Hostinger.

---

# 4. Cloudflare Pages / Workers Deployment Debugging

After DNS-level reasoning, attention moved to Cloudflare deployment.

The deployment logs showed that OpenNext successfully built the app:

```txt
OpenNext build complete.
Worker saved in .open-next/worker.js
```

The deployment also produced a Worker URL:

```txt
https://client-portal-fe.<account>.workers.dev
```

However, Cloudflare also emitted a critical warning:

```txt
Deploying the Worker will override the remote configuration with your local one.
```

The warning showed that remote Dashboard configuration included routes and OIDC environment variables, but the local deployment configuration did not include the same values.

This meant:

```txt
A new deploy could remove remote runtime variables configured manually in Cloudflare Dashboard.
```

That explained why the application later reported invalid or missing frontend OIDC runtime configuration.

---

# 5. Cloudflare Environment Variables Fix

## 5.1 Build Variables

The following build variables were configured:

```txt
NODE_VERSION=22

NEXT_PUBLIC_OIDC_CLIENT_ID=client-portal-fe
NEXT_PUBLIC_OIDC_ISSUER=https://sso.skill-wanderer.com/realms/client-portal
NEXT_PUBLIC_OIDC_REDIRECT_URI=https://client.skill-wanderer.com/auth/callback
NEXT_PUBLIC_OIDC_SILENT_REDIRECT_URI=https://client.skill-wanderer.com/auth/silent-callback
NEXT_PUBLIC_OIDC_LOGOUT_REDIRECT_URI=https://client.skill-wanderer.com/login

OIDC_ISSUER=https://sso.skill-wanderer.com/realms/client-portal
```

## 5.2 Runtime Variables

The same OIDC-related values were also configured as runtime variables:

```txt
NEXT_PUBLIC_OIDC_CLIENT_ID=client-portal-fe
NEXT_PUBLIC_OIDC_ISSUER=https://sso.skill-wanderer.com/realms/client-portal
NEXT_PUBLIC_OIDC_REDIRECT_URI=https://client.skill-wanderer.com/auth/callback
NEXT_PUBLIC_OIDC_SILENT_REDIRECT_URI=https://client.skill-wanderer.com/auth/silent-callback
NEXT_PUBLIC_OIDC_LOGOUT_REDIRECT_URI=https://client.skill-wanderer.com/login
OIDC_ISSUER=https://sso.skill-wanderer.com/realms/client-portal
```

## 5.3 Deploy Command Fix

The deploy command was changed from:

```bash
npx opennextjs-cloudflare deploy
```

to:

```bash
npx opennextjs-cloudflare deploy -- --keep-vars
```

The purpose of `--keep-vars` was to preserve variables configured in Cloudflare Dashboard during Wrangler deployment.

---

# 6. Node.js Version Debugging

At one point, `NODE_VERSION=20` was set because Node.js 20 is often a stable LTS target for Next.js builds.

However, the Cloudflare deployment failed with:

```txt
Wrangler requires at least Node.js v22.0.0.
You are using v20.20.2.
```

The final conclusion was:

```txt
For this project, Node.js 22 is required because the installed Wrangler version requires Node.js >= 22.
```

The final build variable became:

```txt
NODE_VERSION=22
```

or a more specific version:

```txt
NODE_VERSION=22.16.0
```

---

# 7. Keycloak Client Configuration Review

The Keycloak client was inspected.

The relevant configuration was:

```txt
Realm:
client-portal

Client ID:
client-portal-fe

Client authentication:
Off

Standard flow:
On

Direct access grants:
Off

Implicit flow:
Off

PKCE Method:
S256
```

The access settings included:

```txt
Root URL:
https://client.skill-wanderer.com

Home URL:
https://client.skill-wanderer.com/dashboard

Valid redirect URIs:
https://client.skill-wanderer.com/auth/callback
https://client.skill-wanderer.com/auth/silent-callback
https://client.skill-wanderer.com/*

Valid post logout redirect URIs:
https://client.skill-wanderer.com/login
https://client.skill-wanderer.com/*

Web origins:
https://client.skill-wanderer.com
```

This configuration was considered mostly correct for a browser-based public OIDC client using Authorization Code Flow with PKCE.

The remaining issue was therefore not primarily a Keycloak client configuration issue.

---

# 8. Runtime Error: FE_OIDC_ISSUER_INVALID

Cloudflare Observability showed:

```txt
RuntimeConfigError: Invalid frontend runtime configuration
FE_OIDC_ISSUER_INVALID
```

This meant the frontend runtime could not validate its OIDC issuer configuration.

The likely cause was:

```txt
Cloudflare Worker runtime variables were missing or overwritten during deploy.
```

This matched the earlier Wrangler warning about local deployment configuration overwriting remote Dashboard configuration.

After configuring both build and runtime variables and using `--keep-vars`, the issuer problem moved forward to the next layer.

---

# 9. Login Page Loop

After deployment and environment variables were corrected, the login page still flickered between two states:

```txt
Checking sign-in state...
```

and:

```txt
FE_AUTH_BOOTSTRAP_FAILED
```

The displayed message was:

```txt
We could not restore your authenticated session from the browser runtime.
Try signing in again.

We could not restore a valid browser session automatically.
Start a fresh Keycloak sign-in to continue.
```

This indicated that the frontend had moved beyond basic runtime configuration and was now failing during browser auth bootstrap.

---

# 10. Browser Console Investigation

The browser console revealed repeated failed requests:

```txt
POST https://sso.skill-wanderer.com/realms/client-portal/protocol/openid-connect/token 400 (Bad Request)
```

The stack trace showed the repeated path:

```txt
exchangeRefreshToken
useRefreshToken
signinSilent
getUser
setInterval
```

This was the critical clue.

It meant that the frontend was repeatedly attempting silent sign-in or silent token renewal using a stored refresh token.

Keycloak rejected the token refresh with HTTP 400.

The frontend then retried instead of cleaning up stale OIDC browser state.

---

# 11. React Hydration Error

Another browser error appeared:

```txt
Uncaught Error: Minified React error #418
```

React error `#418` indicates a hydration mismatch.

In this project, the likely cause was:

```txt
The server-rendered login page and the first browser-rendered login page did not match.
```

The server could render a neutral login/checking state.

The browser then immediately read localStorage/sessionStorage, found stale OIDC state, attempted silent renew, failed, and rendered a different error state.

That caused the server HTML and client HTML to diverge.

---

# 12. Repository Inspection

The repository was inspected using PowerShell.

The generated build folders existed locally:

```txt
.next/
.open-next/
```

But running:

```powershell
git rm -r --cached .next .open-next
```

returned:

```txt
fatal: pathspec '.next' did not match any files
```

This confirmed:

```txt
.next/ and .open-next/ existed locally but were not tracked by Git.
```

The `.gitignore` was then updated to ensure generated artifacts remain ignored:

```gitignore
.next/
.open-next/
out/
dist/
.vercel/
.wrangler/
```

The temporary `structure.txt` file was removed.

A commit was created:

```bash
git commit -m "chore: ignore generated build artifacts"
```

---

# 13. Source Files Identified

PowerShell was used to locate the relevant source files.

Important files discovered:

```txt
app/layout.tsx
app/(auth)/layout.tsx
app/(auth)/login/page.tsx
app/(portal)/layout.tsx
app/auth/callback/page.tsx
app/auth/silent-callback/page.tsx
components/login/login-page-client.tsx
contexts/AuthContext.tsx
hooks/use-auth.ts
lib/auth.ts
lib/oidc.ts
```

The most important files for the auth loop were:

```txt
lib/oidc.ts
contexts/AuthContext.tsx
components/login/login-page-client.tsx
```

---

# 14. Root Cause in lib/oidc.ts

The OIDC client was configured as:

```ts
automaticSilentRenew: true,
monitorSession: true,
```

The function `tryRestoreOidcUser()` called:

```ts
userManager.signinSilent()
```

when the current user was expired.

The function `refreshOidcUser()` also called:

```ts
userManager.signinSilent()
```

when the current user was expired.

This meant stale refresh tokens were automatically retried.

When Keycloak rejected the refresh token with HTTP 400, the frontend could repeatedly fall into the same flow.

---

# 15. Root Cause in AuthContext.tsx

`AuthContext.tsx` manually started silent renewal:

```ts
userManager.startSilentRenew();
```

It also listened for silent renew errors and converted them into:

```txt
FE_AUTH_BOOTSTRAP_FAILED
```

That made stale or expired browser sessions look like fatal auth bootstrap errors.

The correct behavior should be:

```txt
Stale token / expired browser session
→ cleanup local OIDC state
→ unauthenticated
→ show login button
```

Not:

```txt
Stale token
→ fatal bootstrap failure
→ retry
→ token 400
→ fatal bootstrap failure
→ retry
```

---

# 16. Final Frontend Auth Strategy

The final intended behavior is:

## For login page

```txt
1. Render deterministic initial UI.
2. Restore browser auth state only after client mount.
3. If valid user exists, redirect to /dashboard.
4. If no user exists, show Sign in button.
5. If user is expired or stale, clean browser OIDC state and show Sign in button.
6. Do not automatically call signinSilent on /login.
```

## For stale refresh token

```txt
1. Stop silent renew.
2. Remove stored OIDC user.
3. Clear stale OIDC state.
4. Clear session storage if needed.
5. Set auth state to unauthenticated.
6. Do not record FE_AUTH_BOOTSTRAP_FAILED for normal stale-session cases.
```

## For fatal runtime config failures

Only actual runtime configuration failures should produce fatal frontend errors, such as:

```txt
Missing issuer
Invalid issuer
Missing client ID
Invalid redirect URI
```

---

# 17. Planned / Applied Code-Level Fixes

## 17.1 lib/oidc.ts

The following values must be changed:

```ts
automaticSilentRenew: false,
monitorSession: false,
```

A cleanup function must be added:

```ts
export async function cleanupOidcBrowserState() {
  if (typeof window === "undefined") {
    return;
  }

  let userManager: UserManager | null = null;

  try {
    userManager = getOidcUserManager();
  } catch {
    userManager = null;
  }

  if (userManager) {
    try {
      await userManager.stopSilentRenew();
    } catch {}

    try {
      await userManager.removeUser();
    } catch {}

    try {
      await userManager.clearStaleState();
    } catch {}
  }

  try {
    window.sessionStorage.clear();
  } catch {}
}
```

`tryRestoreOidcUser()` should no longer call `signinSilent()` automatically.

`refreshOidcUser()` should no longer call `signinSilent()` automatically while debugging the login loop.

Expired users should be treated as unauthenticated.

---

## 17.2 AuthContext.tsx

`getLastRuntimeFailure()` should not be read during initial state creation because it reads browser-persisted state and can contribute to hydration mismatch.

This:

```ts
const [lastFailure, setLastFailure] = useState<RuntimeFailure | null>(() =>
  getLastRuntimeFailure()
);
```

should become:

```ts
const [lastFailure, setLastFailure] = useState<RuntimeFailure | null>(null);
```

The provider should not call:

```ts
userManager.startSilentRenew();
```

The silent renew error handler should clean local OIDC state and return to unauthenticated state instead of recording `FE_AUTH_BOOTSTRAP_FAILED`.

---

# 18. Validation Commands

After source-level changes, run:

```powershell
npm run lint
npm run build
```

Then commit:

```powershell
git status
git add lib/oidc.ts contexts/AuthContext.tsx
git commit -m "fix: stop stale oidc silent renew loop"
git push
```

---

# 19. Browser State Cleanup

After deployment, old local browser state must be cleared because stale OIDC tokens may still exist in browser storage.

In DevTools Console:

```js
localStorage.clear();
sessionStorage.clear();

if ("caches" in window) {
  caches.keys().then((keys) => keys.forEach((key) => caches.delete(key)));
}

location.href = "/login";
```

Alternatively, test in a clean Incognito window.

---

# 20. Expected Final Behavior

After the debugging fixes, the login page should behave like this:

```txt
Open /login
→ Checking sign-in state...
→ Continue with SSO
```

If a valid session exists:

```txt
Open /login
→ Checking sign-in state...
→ Redirect to /dashboard
```

If a stale refresh token exists:

```txt
Open /login
→ stale browser token detected
→ local OIDC state cleaned
→ Continue with SSO
```

The page should no longer loop between:

```txt
Checking sign-in state...
FE_AUTH_BOOTSTRAP_FAILED
```

The browser should no longer repeatedly call:

```txt
POST /protocol/openid-connect/token
```

before the user manually starts login.

React hydration error `#418` should disappear because the initial server render and first client render become deterministic.

---

# 21. Lessons Learned

## DNS

NXDOMAIN means the hostname is not resolving at the DNS layer. It should be debugged separately from application build and deployment.

## Cloudflare

Cloudflare Dashboard environment variables can be overwritten by Wrangler deployments if local configuration does not preserve remote variables.

Use:

```bash
npx opennextjs-cloudflare deploy -- --keep-vars
```

or keep the Worker configuration fully codified.

## Node.js

For this project, Node.js 22 is required because the installed Wrangler version requires Node.js >= 22.

## Keycloak

The Keycloak client configuration was mostly correct. The issue was not primarily Keycloak redirect URI configuration.

## Frontend Auth

Silent renew should not be enabled blindly during login bootstrap. Expired or stale browser tokens are normal and should be handled as unauthenticated state.

## React SSR / Hydration

Do not read browser-only auth state during the first render. Browser state should be read after mount to avoid hydration mismatch.

---

# 22. Final Status

The debugging process successfully narrowed the issue from infrastructure-level symptoms to the exact frontend auth-state root cause.

The final root cause was:

```txt
Stale OIDC browser state triggered automatic silent renew.
Keycloak rejected the stale refresh token with HTTP 400.
The frontend treated the failure as FE_AUTH_BOOTSTRAP_FAILED.
Silent renew and auth bootstrap retried repeatedly.
The login UI flickered and React reported hydration mismatch.
```

The final fix direction is:

```txt
Disable automatic silent renew during login bootstrap.
Stop calling signinSilent automatically for expired users.
Clean stale OIDC browser state.
Treat stale sessions as unauthenticated.
Render deterministic login UI.
Only use fatal runtime errors for true configuration failures.
```

This completes the end-to-end debugging narrative for the `client-portal-fe` authentication loop.
