# Security Audit Report — OKX Landing Page
**Date:** June 1, 2026
**Agent:** Security Agent

---

## Audit Summary

| # | Check | Result |
|---|-------|--------|
| 1 | HTTPS enforced (HTTP → HTTPS 301) | ✅ PASS |
| 2 | Mixed content (http:// resources) | ✅ PASS — none found |
| 3 | External links: `rel="noopener noreferrer"` | ✅ FIXED — was `rel="noopener"` only (6 links) |
| 4 | Secrets/credentials in source code | ✅ PASS — none found |
| 5 | Exposed `.git` directory | ✅ PASS — returns 404 |
| 6 | CSP analysis (inline scripts) | 🛡️ MITIGATED — added CSP meta tag |
| 7 | Server headers leaking info | ✅ PASS — only `server: GitHub.com` |
| 8 | Third-party scripts (GA4, etc.) | ✅ PASS — no external scripts loaded |
| 9 | Form security (POST endpoints) | ✅ PASS — no forms on page |
| 10 | SRI on external CDN scripts | ⚠️ INFO — Google Fonts SRI not feasible (dynamic UA-based CSS) |

---

## Issues Found & Fixed

### 1. Referrer Leakage — CRITICAL
**Before:** All 6 external OKX referral links used `rel="noopener"` without `noreferrer`
**Risk:** Referrer header leaked the landing page URL to `web3.okx.com`, exposing UTM parameters and tracking context
**Fix:** Changed to `rel="noopener noreferrer"` on all 6 links
- `/privacy/` and `/terms/` (footer)
- Header CTA (desktop)
- Header CTA (mobile nav)
- Hero section CTA
- CTA section bottom
- Exit-intent popup CTA

### 2. Missing Content-Security-Policy — HIGH
**Before:** No CSP anywhere (GitHub Pages cannot set HTTP headers)
**Risk:** Any XSS vulnerability would have unrestricted script execution
**Fix:** Added `<meta http-equiv="Content-Security-Policy">` with:
- `default-src 'self'` — restrict everything to same-origin
- `script-src 'self' 'unsafe-inline'` — allow self-hosted inline tracking JS
- `style-src 'self' https://fonts.googleapis.com 'unsafe-inline'` — allow Google Fonts
- `font-src https://fonts.gstatic.com` — font delivery
- `img-src 'self' data:` — self-hosted tracking pixel + SVG favicons
- `connect-src 'self'` — no external fetch/XHR
- `base-uri 'self'` — prevent base tag injection
- `form-action 'none'` — no forms allowed
- `frame-ancestors 'none'` — prevent clickjacking

### 3. Missing Referrer-Policy — MEDIUM
**Before:** No referrer policy set at page level
**Fix:** Added `<meta name="referrer" content="strict-origin-when-cross-origin">`
This ensures full URL is sent for same-origin requests, but only origin (no path/query) for cross-origin — complementing `noreferrer` on links.

### 4. Broken Footer Links — MEDIUM
**Before:** `/privacy` and `/terms` in footer returned 404
**Risk:** Users clicking these links would hit GitHub Pages 404 page with poor UX
**Fix:** Created `privacy/index.html` and `terms/index.html` with:
- Full privacy policy (data collection, retention, rights)
- Full terms of service (affiliate disclosure, disclaimers, liability)
- Consistent branding and design

---

## Infrastructure Complement Check

| Check | Status | Notes |
|-------|--------|-------|
| HTTPS enforced (301 redirect) | ✅ | HTTP → HTTPS works correctly |
| CNAME resolution | ✅ | `okx.thealpha-secret.xyz` → `onyxgod777.github.io` (4 A records) |
| Sensitive files exposed | ✅ | All return 404 (`.git`, `.env`, `.htaccess`, `CNAME`, `config.json`, `wp-admin`) |
| Server headers | ✅ | Only `server: GitHub.com` — no version or framework info leaked |
| GitHub Pages Enforce HTTPS | ✅ Implied | Redirect works; GitHub Pages default |
| HSTS header | ⚠️ N/A | GitHub Pages cannot set custom HTTP headers |

---

## Non-Fixable Items (Documented)

1. **Google Fonts SRI:** The Google Fonts CSS response varies by User-Agent (serves different formats: woff2, ttf, etc.), making pre-computed SRI hashes impossible. This is a Google Fonts platform limitation, not a site bug.

2. **HSTS Header:** GitHub Pages does not support custom HTTP headers, so `Strict-Transport-Security` cannot be added. Users who visit the HTTPS version directly are safe. The HTTP→HTTPS redirect mitigates downgrade risk.

3. **GA4:** No Google Analytics is loaded — this is intentional. The page uses a privacy-friendly self-hosted 1x1 tracking pixel at `/track/pixel.gif` with no cookies. This is a security-positive design choice.

---

## Files Created/Modified

| File | Action | Purpose |
|------|--------|---------|
| `index.html` | Modified | Add `noreferrer` to 6 links, CSP meta tag, Referrer-Policy meta tag |
| `privacy/index.html` | Created | Privacy policy page (was returning 404) |
| `terms/index.html` | Created | Terms of service page (was returning 404) |

---

## Retrospective

- The biggest security gap was the referrer leakage — `noopener` prevents `window.opener` access but does NOT block the `Referer` header. This is a common misconception.
- The CSP meta tag approach is a viable fallback for GitHub Pages (which cannot set HTTP headers). While `unsafe-inline` is required for the self-hosted tracking script, the `form-action 'none'` and `frame-ancestors 'none'` directives provide strong clickjacking and form injection protection.
- The self-hosted tracking pixel avoids third-party script risks entirely — no GA4, no Facebook Pixel, no external CDN scripts.
- /privacy and /terms pages were needed both for compliance and to fix broken UX (404 errors on footer links).
