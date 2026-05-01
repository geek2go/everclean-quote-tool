# EverClean Quote Tool — Repo Guide

> **What this repo is:** the deploy source for a single-file static web tool. It is the *code* repo. The *spec, decisions, and issue history* live in a separate (private) consulting folder; pointers below.

## Ownership & roles

- **Tool owner:** EverClean Home Services LLC — Brandon & Angela Erskine (Brainerd Lakes Area, MN)
- **Maintainer:** Laura (Geek2Go) — consulting engagement
- **GitHub repo:** under the `geek2go` GitHub account (Laura maintains)
- **Cloud / API key billing:** under the `evercleanlakesarea.com` Google Workspace org (client owns the API costs)

That split is intentional. Don't move the API key out from under EverClean's Cloud project.

## What this tool is

Internal-only mobile web tool. Brandon/Angela open it on a phone bookmark during a customer call, fill in scope (property type, home size, frequency, scope traps, add-ons), tap **Generate Quote**, copy the formatted plain-text quote to clipboard, paste it into Quo (their texting platform). Customer never sees the tool. 15-minute call-to-quote SLA.

Replaces verbal phone-quoting entirely. Closes the YES on the call; closes the PRICE in writing.

## Architecture (the only file that matters)

**`index.html`** — everything. CSS + JS + HTML + JSON config, all embedded. No build step. No bundler. No framework. Editable in any text editor.

The `CONFIG` object near the top of the `<script>` block holds:
- `googleMapsApiKey` — Places API key (referrer-restricted)
- `minimumQuote` — hard floor ($160)
- `rushSurcharges` — % multipliers by scheduling option
- `rateCard` — per service-type, per home-size, per frequency
- `homeSizes` — radio button labels + sqft ranges
- `addOns` — name, price, perUnit/qty, optional clarifying note
- `scopeSummaries` — customer-facing "Includes:" copy for each service (derived from evercleanlakesarea.com/services/ — keep verbatim)
- `prepRequirements` — customer-side conditions for Move-In/Move-Out/STR scope to apply
- `business` — name, phone, website, quote validity days

**Edit pricing without touching code:** change `CONFIG` values, push, Pages redeploys.

## Deploy workflow

GitHub Pages serves `index.html` from the `main` branch root.

```
edit index.html → git push → Pages rebuilds in ~60s → live
```

Live URL: <https://geek2go.github.io/everclean-quote-tool/>

No CI, no staging, no preview branch. The tool is internal-only and small enough that direct-to-main is appropriate. Acceptance tests (below) are run by hand against the calculation engine.

## Local development

```bash
npx http-server . -p 8765 -c-1
# then open http://localhost:8765/
```

http-server (or any static server) is required — opening `index.html` from `file://` will not load the Google Maps API (referrer mismatch). Localhost is allow-listed in the API key restrictions.

## Acceptance tests (must continue to pass)

Eight tests, defined in the consulting folder's `BUILD-PLAN.md` §9. Quick summary:

| # | Scenario | Expected |
|---|---|---|
| 1 | STR Turnover Small same-day, linens×2 + laundry×2 charged | $313 + STR prep section |
| 2 | Resort/Multi-cabin | Site-walk template, no price |
| 3 | XL biweekly Standard, INCLUDE bundle + $40 discount | $275, $345 bundled value, no prep section |
| 4 | Studio Weekly | $160 (minimum floor enforced) |
| 5 | Estate home size | Site-walk template |
| 6 | Hazardous = Yes | Site-walk template |
| 7 | $40 discount with blank reason | Validation error |
| 8 | Standard Medium one-time same-day | $281 (rush math) |

After any change to the calculation engine or templates, run them mentally before pushing.

## API key safety

The `googleMapsApiKey` in `CONFIG` is **HTTP-referrer-restricted** in the EverClean Cloud project to:
- `https://geek2go.github.io/everclean-quote-tool/*`
- `http://localhost/*` (for local dev)

The key being in public source is **expected and safe** — *only because of the referrer lock*. Never disable that restriction. If the key needs rotating, generate a new one in EverClean's Cloud Console, update `CONFIG.googleMapsApiKey`, push.

## Brand & UI references (private)

This repo intentionally does NOT include design refs. They live in the consulting folder:

- `_consulting/CLAUDE.md` — full project guide, brand colors, voice, services
- `_consulting/DESIGN.md` — Apple-inspired UI patterns reference
- `_consulting/pricing/quote-tool-spec.md` — design spec
- `_consulting/pricing/BUILD-PLAN.md` — implementation plan + acceptance tests
- `_consulting/DECISIONS.md` — pricing & policy decisions
- `_consulting/ISSUES.md` — open issue tracker

If you (an agent) are working in this repo without access to the consulting folder, ask Laura before making style decisions or rate-card changes. Don't paraphrase scope summaries — they're verbatim from the website.

**Brand quick reference (in case you can't reach the consulting folder):**

| Token | Value |
|---|---|
| EC Blue | `#00559C` |
| EC Green | `#68B578` |
| Off-white | `#F8F9FA` |
| Heading font | Poppins 700 |
| Body font | Roboto |
| Brand voice | Plain, honest, Minnesotan — no fluff. Trusted neighbor, not a corporate brochure. |

UI uses Apple-pattern UX (pill CTAs, hairline borders, 17px body, 8px-base spacing) on EverClean brand tokens. Don't introduce a second accent or change fonts.

## Hard rules

- ✗ **Don't change rate-card values** without explicit Laura sign-off and a corresponding entry in `_consulting/DECISIONS.md`.
- ✗ **Don't disable the API key referrer restriction.** Ever.
- ✗ **Don't add a second API key, secret, or credential** to `CONFIG` or anywhere in the code without flagging it.
- ✗ **Don't paraphrase `scopeSummaries`** — they're derived from the live website. If website copy changes, update here verbatim.
- ✗ **Don't add a build step, framework, or bundler.** This file is supposed to be editable by Laura in any text editor.
- ✗ **Don't add localStorage / persistence / analytics** without confirming with Laura. The original spec calls out "ephemeral by design — no historical data leaks" as a feature; reversing that is a real product decision.

## Future enhancements (parked)

Out of scope for v1, listed for context:

- Save/load drafts (localStorage)
- Quote log to a Google Sheet (Apps Script web app endpoint)
- ZenMaid booking handoff
- Mixed-frequency contracts (e.g., biweekly summer + monthly winter)
- Commercial / multi-cabin auto-pricing (currently → site walk)
- Customer-facing version

If a request lands in any of these areas, surface the tradeoffs before implementing — most have privacy or scope implications.
