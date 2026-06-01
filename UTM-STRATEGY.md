# OKX Referral Landing Page — UTM Parameter Strategy

## Overview

This document defines the UTM parameter structure for tracking traffic sources to
the OKX landing page (https://thealpha-secret.xyz/) and the referral link
(https://web3.okx.com/join/HERMES). All OKX referral links on the page are
automatically enriched with UTM parameters via JavaScript on page load.

---

## UTM Parameters Used

| Parameter      | Description                        | Example                 |
|----------------|------------------------------------|-------------------------|
| `utm_source`   | Traffic origin                     | twitter, telegram, google|
| `utm_medium`   | Marketing channel                  | social, email, banner   |
| `utm_campaign` | Campaign name                      | hermes-referral         |
| `utm_content`  | Specific element/link              | hero-cta, exit-popup    |
| `utm_term`     | (Reserved for keywords/search)     | crypto-exchange         |

---

## Traffic Source → UTM Mapping

### Organic / Direct
- Source: thealpha-secret.xyz
- `utm_source=direct`
- `utm_medium=none`
- Used when no query params are present (default)

### Social Media
| Platform   | URL to use                                        |
|------------|---------------------------------------------------|
| Twitter/X  | ?utm_source=twitter&utm_medium=social&utm_campaign=hermes-referral&utm_content={post_id} |
| Telegram   | ?utm_source=telegram&utm_medium=social&utm_campaign=hermes-referral&utm_content={group} |
| Reddit     | ?utm_source=reddit&utm_medium=social&utm_campaign=hermes-referral&utm_content={subreddit} |
| Discord    | ?utm_source=discord&utm_medium=social&utm_campaign=hermes-referral&utm_content={server} |

### Email / Newsletter
- `utm_source=newsletter&utm_medium=email&utm_campaign=hermes-referral&utm_content={issue_number}`

### Paid / Ads
- `utm_source=google&utm_medium=cpc&utm_campaign=hermes-referral&utm_content={ad_group}`
- `utm_source=twitter&utm_medium=paid&utm_campaign=hermes-referral&utm_content={ad_id}`

### Content / Blog
- `utm_source=medium&utm_medium=article&utm_campaign=hermes-referral&utm_content={topic}`
- `utm_source=blog&utm_medium=organic&utm_campaign=hermes-referral&utm_content={post_slug}`

### Referral / Partner
- `utm_source={partner_name}&utm_medium=referral&utm_campaign=hermes-referral&utm_content={partner_id}`

---

## Internal Link Attribution

Each OKX referral link on the page is tagged with `data-utm-medium` for
analytics differentiation:

| Link Location        | data-utm-medium    | data-utm-source     |
|----------------------|--------------------|---------------------|
| Header "Get Started" | header             | (inherits from URL) |
| Hero CTA button      | hero               | (inherits from URL) |
| Feature card CTA     | features           | (inherits from URL) |
| Exit-intent popup    | popup              | exit-intent         |
| Footer link          | footer             | (inherits from URL) |
| Mobile nav CTA       | header-mobile      | (inherits from URL) |

---

## Tracking Pixel Events

The self-hosted 1x1 tracking pixel at `/track/pixel.gif` fires GET requests with:

| Event Type          | Detail                           | Fires When                         |
|---------------------|----------------------------------|-------------------------------------|
| pageview            | landing-page                     | On page load                       |
| conversion-click    | hero-cta / cta-bottom / popup    | User clicks any OKX referral link  |
| exit-intent         | triggered / dismissed            | Exit-intent popup shown/dismissed  |
| ab-test             | variant-a / variant-b            | A/B test variant assigned           |

All pixel requests include: `type`, `source`, `medium`, `campaign`, `session`,
`ts`, `detail`, and a random cache-buster.

---

## Setting Up the Tracking Pixel Endpoint

### Option 1: Self-hosted (recommended)
Create `/track/pixel.gif` as a 1x1 transparent GIF served by your web server.
A PHP/Node.js endpoint at `/track/pixel.gif` can log events to a database or
file while returning the GIF.

### Option 2: Analytics service
Replace PIXEL_ENDPOINT in the JavaScript with a service like:
- Plausible: `https://plausible.io/js/script.js`
- Simple Analytics: `https://scripts.simpleanalyticscdn.com/latest.js`
- Fathom: `https://cdn.usefathom.com/script.js`

### Option 3: Google Analytics 4
Add the GA4 gtag snippet and track events via gtag('event', ...) calls.

---

## A/B Testing

The page includes a URL-based A/B test system:
- Append `?test=a` or `?test=b` to the URL
- The variant is stored in sessionStorage
- HTML gets `data-ab-test="a"` or `data-ab-test="b"` attribute
- Use CSS rules like: `[data-ab-test="a"] .hero h1 { ... }`

For Google Optimize integration (commented out in the code), uncomment the
`ga('require', 'GTM-XXXXXXX')` line and add your Optimize container ID.

---

## Attribution Window

- **Click-through**: 30-day cookie/session-based attribution
- **View-through**: 24-hour window for impression-based attribution
- **Cross-device**: Use the same UTM source across all shared links
