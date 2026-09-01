# Brevo Ecommerce Free Tools

8 free tools + a hub page for brevo.com/ecommerce/tools/. Built as standalone single-file
HTML pages: no framework, no build step at deploy time, no backend, no secrets.

Live demo: https://farooqchisty-hub.github.io/creator-pools/brevo-tools/

## How this was built

- `build/build.py` assembles each page from shared Brevo chrome (header, breadcrumb, hero,
  signup banners, related-tools grid, FAQ, footer, all CSS) plus one content module per
  tool (`build/fragments/t2.py` ... `t8.py`: copy, tool markup, engine JS, FAQs).
  Run `python3 build.py` to regenerate `dist/`.
- Design system: Brevo Naos tokens (forest green 800 #006a43, cream 200 #faf5e3, sun
  yellow CTAs), official wordmark SVG from the product, Tomato Grotesk embedded as
  base64 data URIs, Inter via Google Fonts.
- Every page ships JSON-LD (SoftwareApplication, BreadcrumbList, FAQPage) and its
  keyword-cluster copy per the SEO plan. No em dashes anywhere (brand rule).
- Tool 1 (deliverability checker) predates the build system and is maintained as a plain
  HTML file; everything else regenerates from fragments.

## What each tool runs on (all client-side)

| Tool | Engine | Data that needs maintenance |
| --- | --- | --- |
| email-deliverability-checker | DNS-over-HTTPS (dns.google, cloudflare fallback), RDAP for domain age, Spamhaus DBL | DKIM selector list (16 selectors) |
| abandoned-cart-calculator | Benchmark math (abandonment 70% Baymard; recovery 2.2 to 4.5% by industry; SMS +45%, push +15% uplift) | Benchmark table, review yearly |
| subject-line-tester | Rule engine (length, ~60 spam triggers by category, caps, emoji, punctuation) + per-client truncation budgets | Spam word list, client char budgets |
| sms-whatsapp-cost-calculator | Embedded rate table (22 markets), GSM-7 vs Unicode segment counter | RATES table in t4.py: VERIFY against Meta rate card and Brevo SMS pricing before launch, refresh quarterly |
| email-pricing-comparison | Tier tables for 7 platforms, linear interpolation between public price points | TIERS table in t5.py: VERIFY before launch, refresh quarterly |
| email-list-health-checker | Local CSV/paste parsing, syntax RFC-lite regex, 46 disposable domains, 27 role prefixes, typo map, gibberish heuristics, optional MX check via DoH (60-domain cap) | Disposable list (expand to a full ~4k list at launch) |
| ecommerce-email-grader | DNS auth check + platform detection heuristics + 12-question weighted self-assessment (lite version) | Question weights |
| bfcm-calendar-generator | Dated Q4-2026 send template, market shipping cutoffs, benchmark revenue model, client-side CSV + ICS export | Dates are 2026-specific: regenerate for 2027 |

## Deploying

1. Copy each `dist/<slug>/index.html` to `brevo.com/ecommerce/tools/<slug>/` and
   `dist/index.html` to `/ecommerce/tools/`. Any static host works; pages are relative-linked
   (`../<slug>/`), so keep the folder structure.
2. Launch checklist per page:
   - Email gates: search for `PRODUCTION WIRING` comments. Replace the demo click handlers
     with a Brevo hosted form POST (no API key in the page), and send the gated PDF/CSV via a
     transactional template.
   - Replace placeholder `#` hrefs in the top nav, banners and footer with real brevo.com URLs.
   - Add the site analytics snippet and cookie consent per brevo.com standards.
   - Have SEO verify the rate/pricing tables (t4, t5) and stamp the as-of date in the copy.
3. No CSP requirements beyond allowing `fonts.googleapis.com`/`gstatic`, `dns.google`,
   `cloudflare-dns.com` and `rdap.org` as connect/style sources.

## Backend features deliberately deferred (phase 2)

All three share the same infra (suggest one Cloudflare Worker project + Email Routing):

1. **Test-email mode for the deliverability checker**: a receive mailbox
   (check-abc123@tools.brevo.com) that inspects a real message: sending IP, rDNS, live DKIM
   signature validation, SpamAssassin-style content score. This is how mail-tester works.
2. **SMTP-level verification for the list checker**: mailbox-existence pings, catch-all
   detection and true spam-trap screening (needs server IPs with reputation).
3. **Full automated program grader**: crawler that detects popups and platform, subscribes a
   test address, and measures real welcome/cart timing via the same inbox infra. Turns the
   lite self-assessment into HubSpot-grader-class automation, per the SEO plan.

Smaller worthwhile upgrades: a first-party DoH proxy on a Worker (adds query analytics and
rate limiting), server-rendered PDF for gated reports, AI subject line variants behind a
Worker (currently linked out to brevo.com/ai-email-generator), and a monitoring/alerts
feature which requires accounts.

## QA that was run

Functional DOM tests per tool in Chrome (math spot-verified by hand: cart model, pricing
interpolation, GSM-7/Unicode segment switch, list issue detection, grader weighting, calendar
dates and exports), live DNS engines tested against real domains, full-page visual review of
hub and tool pages, mojibake/charset regression covered (`<meta charset>` first line),
em-dash lint in the build, all pages relative-linked and verified at 200 on GitHub Pages.
