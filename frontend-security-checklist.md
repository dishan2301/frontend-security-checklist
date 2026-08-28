# Frontend Security Checklist — Bank-Locker Standard

Frontend-only doesn't mean low-risk. The browser is the most exposed part of your stack — attackers touch it directly. Treat every item below as non-negotiable, not "nice to have."

---

## 1. Transport & Headers

- [ ] **HTTPS everywhere** — no mixed content (HTTP images/scripts on an HTTPS page). Redirect all HTTP → HTTPS.
- [ ] **HSTS** (`Strict-Transport-Security`) — force browsers to always use HTTPS, include `preload` and `includeSubDomains`.
- [ ] **Content-Security-Policy (CSP)** — the single biggest defense against XSS. Whitelist only the domains/scripts you actually use. Avoid `unsafe-inline` and `unsafe-eval`.
- [ ] **X-Content-Type-Options: nosniff** — stops browsers guessing file types.
- [ ] **X-Frame-Options: DENY** or `frame-ancestors 'none'` in CSP — prevents clickjacking (your site loaded in a hidden iframe).
- [ ] **Referrer-Policy: strict-origin-when-cross-origin** — don't leak full URLs (which may contain tokens) to third-party sites.
- [ ] **Permissions-Policy** — explicitly disable camera/mic/geolocation/etc. unless you need them.

## 2. Cross-Site Scripting (XSS)

- [ ] Never insert user input into the DOM with `innerHTML`, `dangerouslySetInnerHTML` (React), or `v-html` (Vue) without sanitizing (use DOMPurify).
- [ ] Escape/encode all dynamic content rendered into HTML, attributes, URLs, and JS contexts — each context needs different encoding.
- [ ] Avoid `eval()`, `new Function()`, and dynamic `setTimeout("string")`.
- [ ] Sanitize and validate anything coming from URL params, query strings, or `localStorage`/`sessionStorage` before rendering.
- [ ] If you allow rich text (comments, bios), sanitize on both input and render — never trust "it was cleaned earlier."

## 3. Authentication & Session Handling (frontend side)

- [ ] Never store JWTs, session tokens, or passwords in `localStorage` or `sessionStorage` — both are readable by any injected script (XSS = instant token theft). Prefer **HttpOnly, Secure, SameSite=Strict cookies** set by the backend.
- [ ] If you must hold a token client-side briefly (e.g., in memory/JS variable), never persist it, and clear it on tab close/logout.
- [ ] Implement auto-logout / token expiry handling in the UI — don't let stale sessions linger silently.
- [ ] Use `SameSite=Strict` or `Lax` cookies to reduce CSRF exposure.
- [ ] Never expose password fields with autocomplete on shared/public devices without a way to disable it; enforce strong password rules client-side as a UX aid (never as the only defense).

## 4. CSRF Protection

- [ ] Even "frontend only," if you talk to any backend/API that changes state (login, forms, payments), ensure CSRF tokens are used, or rely on `SameSite` cookies plus proper CORS.
- [ ] Never perform state-changing actions (delete, transfer, update) via simple `GET` requests.

## 5. CORS

- [ ] If your frontend calls APIs, ensure the API's CORS policy is locked to your specific domain(s) — never trust `Access-Control-Allow-Origin: *` for anything involving auth or sensitive data.
- [ ] Don't blindly send credentials (`credentials: 'include'`) to third-party origins.

## 6. Dependency & Supply Chain Security

- [ ] Run `npm audit` / `yarn audit` regularly, and fix high/critical vulnerabilities immediately.
- [ ] Pin dependency versions (lockfiles committed) — don't let a compromised patch version slip in silently.
- [ ] Use **Subresource Integrity (SRI)** hashes (`integrity="sha384-..."`) on any script/style loaded from a CDN.
- [ ] Minimize third-party scripts (analytics, chat widgets, ad tags) — each one is a potential supply-chain attack vector. Audit what data they can access.
- [ ] Avoid abandoned/unmaintained npm packages; check last-updated dates and open CVEs before adding a dependency.

## 7. Sensitive Data Exposure

- [ ] Never hardcode API keys, secrets, or credentials in frontend JS bundles — anything shipped to the browser is public, full stop.
- [ ] Check your build output (`dist`/`build` folder) for accidentally bundled `.env` values, comments with internal URLs, or debug info.
- [ ] Disable source maps in production, or restrict access to them — they can reveal your full source structure.
- [ ] Strip verbose error messages, stack traces, and debug logs from production builds.
- [ ] Don't expose internal API endpoints, admin routes, or feature flags in client-side code/config that a user could discover via dev tools.

## 8. Input Validation (client-side, as UX — never as the only gate)

- [ ] Validate all form inputs client-side for UX, but always re-validate server-side — client validation is trivially bypassed.
- [ ] Enforce file type/size checks before upload, but treat them as advisory only (attacker can bypass the browser entirely).
- [ ] Sanitize file names and paths if displaying uploaded content.

## 9. Clickjacking & UI Redress

- [ ] `X-Frame-Options` / `frame-ancestors` set as above.
- [ ] For sensitive actions (payments, account changes), consider re-confirmation steps that are harder to spoof via overlay attacks.

## 10. Third-Party Scripts & Widgets

- [ ] Load third-party scripts with `async`/`defer` and SRI where possible.
- [ ] Sandbox untrusted embeds (chat widgets, iframes) using the `sandbox` attribute.
- [ ] Regularly review what tracking/analytics scripts have access to (cookies, DOM, form data).

## 11. Build & Deployment

- [ ] Ensure `.env`, `.git`, config files, and backup files are never publicly accessible on the deployed server (check `/`, `/.git/`, `/.env` directly).
- [ ] Set proper cache-control headers so sensitive pages (dashboards, account pages) aren't cached by browsers/proxies.
- [ ] Use a Web Application Firewall (WAF) or CDN-level protection (Cloudflare, etc.) in front of the site if possible.
- [ ] Rate-limit or protect public-facing forms (contact, login, signup) against bots — even if enforcement is server-side, the frontend should support CAPTCHA/verification UI.

## 12. Regular Practices (ongoing, not one-time)

- [ ] Run automated security scans (OWASP ZAP, Lighthouse security audit, Mozilla Observatory) before each major release.
- [ ] Do a manual pass with browser dev tools: check Network tab for leaked tokens, Application tab for what's stored client-side, and Console for exposed debug data.
- [ ] Keep a changelog of security-relevant changes so you can audit what changed and when.
- [ ] Review this checklist every time you add a new third-party integration or major feature — new attack surface = new risk.

---

**Bottom line:** a "frontend-only" site still touches auth, forms, third-party scripts, and data rendering — all classic attack surfaces. The two most common ways frontend-only sites get breached are (1) tokens stored in `localStorage` getting stolen via XSS, and (2) missing/misconfigured CSP headers. Fix those two first if you do nothing else, then work through the rest of this list.
