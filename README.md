# 🔒 Frontend Security Checklist

A practical, no-shortcuts security checklist for frontend-only websites and web apps — covering transport security, XSS prevention, safe token/session handling, CORS, dependency/supply-chain risks, sensitive data exposure, and deployment hardening.

Built to catch the real-world mistakes that lead to breaches, not just tick boxes.

![Security](https://img.shields.io/badge/security-checklist-critical)
![Status](https://img.shields.io/badge/status-active-brightgreen)
![License](https://img.shields.io/badge/license-MIT-blue)

---

## 📋 Table of Contents

1. [Transport & Headers](#1-transport--headers)
2. [Cross-Site Scripting (XSS)](#2-cross-site-scripting-xss)
3. [Authentication & Session Handling](#3-authentication--session-handling-frontend-side)
4. [CSRF Protection](#4-csrf-protection)
5. [CORS](#5-cors)
6. [Dependency & Supply Chain Security](#6-dependency--supply-chain-security)
7. [Sensitive Data Exposure](#7-sensitive-data-exposure)
8. [Input Validation](#8-input-validation-client-side-as-ux--never-as-the-only-gate)
9. [Clickjacking & UI Redress](#9-clickjacking--ui-redress)
10. [Third-Party Scripts & Widgets](#10-third-party-scripts--widgets)
11. [Build & Deployment](#11-build--deployment)
12. [Regular Practices](#12-regular-practices-ongoing-not-one-time)
13. [Top Priority Fixes](#-if-you-only-fix-two-things)

---

## 1. Transport & Headers

- [ ] **HTTPS everywhere** — no mixed content (HTTP images/scripts on an HTTPS page). Redirect all HTTP → HTTPS.
- [ ] **HSTS** (`Strict-Transport-Security`) — force browsers to always use HTTPS, include `preload` and `includeSubDomains`.
- [ ] **Content-Security-Policy (CSP)** — the single biggest defense against XSS. Whitelist only the domains/scripts you actually use. Avoid `unsafe-inline` and `unsafe-eval`.
- [ ] **X-Content-Type-Options: nosniff** — stops browsers guessing file types.
- [ ] **X-Frame-Options: DENY** or `frame-ancestors 'none'` in CSP — prevents clickjacking.
- [ ] **Referrer-Policy: strict-origin-when-cross-origin** — don't leak full URLs (which may contain tokens) to third-party sites.
- [ ] **Permissions-Policy** — explicitly disable camera/mic/geolocation/etc. unless needed.

## 2. Cross-Site Scripting (XSS)

- [ ] Never insert user input into the DOM with `innerHTML`, `dangerouslySetInnerHTML` (React), or `v-html` (Vue) without sanitizing (use DOMPurify).
- [ ] Escape/encode all dynamic content rendered into HTML, attributes, URLs, and JS contexts.
- [ ] Avoid `eval()`, `new Function()`, and dynamic `setTimeout("string")`.
- [ ] Sanitize and validate anything from URL params, query strings, or `localStorage`/`sessionStorage` before rendering.
- [ ] Sanitize rich text (comments, bios) on both input and render.

## 3. Authentication & Session Handling (frontend side)

- [ ] Never store JWTs, session tokens, or passwords in `localStorage`/`sessionStorage` — both are readable by any injected script. Prefer **HttpOnly, Secure, SameSite=Strict cookies** set by the backend.
- [ ] If a token must live client-side briefly, never persist it, and clear it on tab close/logout.
- [ ] Implement auto-logout / token expiry handling in the UI.
- [ ] Use `SameSite=Strict` or `Lax` cookies to reduce CSRF exposure.
- [ ] Enforce strong password rules client-side as a UX aid only — never as the sole defense.

## 4. CSRF Protection

- [ ] Ensure CSRF tokens are used for any state-changing request, or rely on `SameSite` cookies plus proper CORS.
- [ ] Never perform state-changing actions (delete, transfer, update) via simple `GET` requests.

## 5. CORS

- [ ] Lock API CORS policy to your specific domain(s) — never `Access-Control-Allow-Origin: *` for anything involving auth or sensitive data.
- [ ] Don't send credentials (`credentials: 'include'`) to third-party origins.

## 6. Dependency & Supply Chain Security

- [ ] Run `npm audit` / `yarn audit` regularly; fix high/critical vulnerabilities immediately.
- [ ] Pin dependency versions (commit lockfiles).
- [ ] Use **Subresource Integrity (SRI)** hashes (`integrity="sha384-..."`) on CDN-loaded scripts/styles.
- [ ] Minimize third-party scripts (analytics, chat widgets, ad tags) — audit what data they can access.
- [ ] Avoid abandoned/unmaintained npm packages; check last-updated dates and open CVEs.

## 7. Sensitive Data Exposure

- [ ] Never hardcode API keys, secrets, or credentials in frontend JS — anything shipped to the browser is public.
- [ ] Check build output (`dist`/`build`) for accidentally bundled `.env` values or debug info.
- [ ] Disable source maps in production, or restrict access to them.
- [ ] Strip verbose error messages, stack traces, and debug logs from production builds.
- [ ] Don't expose internal API endpoints, admin routes, or feature flags client-side.

## 8. Input Validation (client-side, as UX — never as the only gate)

- [ ] Validate all form inputs client-side for UX, always re-validate server-side.
- [ ] Enforce file type/size checks before upload — treat as advisory only.
- [ ] Sanitize file names and paths if displaying uploaded content.

## 9. Clickjacking & UI Redress

- [ ] `X-Frame-Options` / `frame-ancestors` set as above.
- [ ] Add re-confirmation steps for sensitive actions (payments, account changes).

## 10. Third-Party Scripts & Widgets

- [ ] Load third-party scripts with `async`/`defer` and SRI where possible.
- [ ] Sandbox untrusted embeds using the `sandbox` attribute.
- [ ] Regularly review what tracking/analytics scripts can access (cookies, DOM, form data).

## 11. Build & Deployment

- [ ] Ensure `.env`, `.git`, config, and backup files are never publicly accessible (`/`, `/.git/`, `/.env`).
- [ ] Set proper cache-control headers so sensitive pages aren't cached by browsers/proxies.
- [ ] Use a WAF or CDN-level protection (e.g., Cloudflare) in front of the site.
- [ ] Rate-limit/protect public-facing forms against bots — support CAPTCHA/verification in the UI.

## 12. Regular Practices (ongoing, not one-time)

- [ ] Run automated security scans (OWASP ZAP, Lighthouse security audit, Mozilla Observatory) before each release.
- [ ] Manually check dev tools: Network tab for leaked tokens, Application tab for client-side storage, Console for exposed debug data.
- [ ] Keep a changelog of security-relevant changes.
- [ ] Re-review this checklist whenever a new third-party integration or major feature is added.

---

## 🚨 If You Only Fix Two Things

1. **Never store auth tokens in `localStorage`/`sessionStorage`.** One XSS bug = instant token theft. Use HttpOnly cookies instead.
2. **Set a real Content-Security-Policy.** Most frontend breaches trace back to a missing or too-loose CSP.

---

## 🤝 Contributing

Found something missing or outdated? PRs and issues welcome — this checklist should evolve as attack techniques do.

## 📄 License

MIT
