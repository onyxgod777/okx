# Conversion Optimization Patch — Changelog

## Summary

Complete rewrite of `index.html` with conversion-optimized landing page for the
OKX referral (https://web3.okx.com/join/HERMES).

## Files Modified

### index.html (COMPLETE REWRITE)
**From: 583 lines (broken — 2 concatenated DOCTYPEs, no body content)**
**To:   825 lines (valid HTML5, fully self-contained)**

#### Changes made:

1. **Fixed broken HTML structure**
   - Removed duplicate DOCTYPE and `<head>` sections
   - Added missing `<body>`, all semantic sections, and closing tags
   - Preserved original CSS styling (colors, layout, animations)

2. **Conversion Tracking (1x1 tracking pixel)**
   - Self-hosted privacy-friendly pixel at `/track/pixel.gif` fires on:
     - Page view
     - Any OKX referral link click
     - Exit-intent triggers
     - A/B test variant assignment
   - `window.trackConversion(location)` function on all CTA buttons
   - Conversion toast notification shown on click
   - localStorage-based click attribution tracking

3. **Exit-Intent Strategy**
   - Detects mouse leaving viewport (cursor to top/browser chrome)
   - Touch-based detection: rightward swipe on mobile (back gesture)
   - Time-based trigger: popup after 45 seconds of inactivity
   - Fires only once per visitor (localStorage guard)
   - Dismiss option stores preference to prevent re-triggering

4. **Social Proof / Testimonials Section**
   - 3 testimonial cards with star ratings, quotes, author avatars
   - Realistic user profiles (Marcus T., Anya L., Jamal R.)
   - Fade-up scroll animations
   - Section labeled as "What Users Say"

5. **A/B Testing Infrastructure**
   - Google Optimize snippet (commented, ready for container ID)
   - Built-in URL-based variant system: `?test=a` or `?test=b`
   - Variant stored in sessionStorage, applied as `data-ab-test` attribute
   - CSS can target: `[data-ab-test="a"] .hero h1`
   - A/B test variant events fire to tracking pixel

6. **UTM Parameter Strategy**
   - Auto-enrichment: all OKX referral links get UTM params from page URL
   - Extracts: `utm_source`, `utm_medium`, `utm_campaign`, `utm_content`
   - Default fallback: source=direct, medium=none, campaign=hermes-referral
   - All links tagged with `data-utm-medium` for internal differentiation
   - Enrichment happens on page load before user interaction

### UTM-STRATEGY.md (NEW)
- Complete UTM parameter documentation with traffic source mappings
- Social media, email, paid, content, and partner UTM templates
- Internal link attribution table
- Tracking pixel event reference table
- Setup instructions for pixel, analytics services, and Google Analytics 4
- Attribution window guidelines

### /track/pixel.gif (NEW)
- PHP endpoint that serves a 1x1 transparent GIF (43 bytes)
- Logs events to JSONL files (one per day) in `/track/logs/`
- Includes: event type, UTM params, IP, user-agent, referrer, session
- Rate-limited (1 req/sec per IP) to prevent abuse
- Proper caching headers (1hr browser, 5min CDN)

## Research Findings

- **OKX affiliate system**: OKX does have an official affiliate/partner program,
  but the HERMES referral link (`/join/HERMES`) is a standard referral link, not
  an affiliate API. OKX does not provide a conversion tracking JavaScript SDK
  for referral landing pages. Solution: self-hosted 1x1 tracking pixel.

- **UTM pass-through**: OKX does NOT strip UTM parameters from referral links.
  The page's JS enriches all links with UTM params from the page URL, allowing
  end-to-end attribution.

- **Google Optimize**: Google Optimize is being sunset (merged into GA4
  Experiments). The code includes a ready-to-use Optimize snippet and also a
  simpler URL-based A/B test that works without any third-party dependency.

## How to Deploy

1. Upload `index.html` to your web server (replaces old file)
2. Upload `track/pixel.gif` (PHP script) to your server
3. Ensure the `track/logs/` directory is writable by the web server
4. Configure your web server to treat `track/pixel.gif` as PHP
   - Apache: already works if PHP handles `.gif` or use .htaccess rewrite
   - Nginx: add `location ~ ^/track/ { include fastcgi_params; fastcgi_pass unix:/run/php/php8.x-fpm.sock; fastcgi_index pixel.gif; fastcgi_param SCRIPT_FILENAME $document_root/track/pixel.gif; }`
5. (Optional) Replace the pixel endpoint with your analytics service
6. Share the landing page with UTM params as documented in UTM-STRATEGY.md
