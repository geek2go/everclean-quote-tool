# EverClean Quote Tool

Internal-only mobile web tool used by EverClean Lakes Area (Brandon & Angela Erskine) to capture quote details during phone calls and generate a customer-ready quote text/email in plain text. Output is copy-pasted into Quo (their texting platform) within a 15-minute call-to-quote SLA.

**Live:** https://geek2go.github.io/everclean-quote-tool/

## What this is

A single-file HTML tool. Brandon/Angela open it on a phone bookmark during a call, fill in scope details (property type, home size, frequency, scope traps, add-ons), tap **Generate Quote**, and copy the formatted text to clipboard. The tool replaces verbal phone-quoting entirely.

## Architecture

- Single file: `index.html` — embedded CSS + JS + JSON config, no build step.
- All pricing in a `CONFIG` object at the top of the `<script>` block. Editable in any text editor when rates change.
- No backend. No persistence. Each session is fresh.
- Hosted as a static site on GitHub Pages.

## How to update pricing

1. Edit `index.html` → find the `const CONFIG = { ... }` block near the top of the script.
2. Change rate-card values, add-on prices, surcharges, or business info as needed.
3. Commit and push — GitHub Pages auto-deploys within ~1 minute.

## Address autocomplete

Uses Google Places API. The API key sits in `CONFIG.googleMapsApiKey`. The key is **HTTP-referrer-restricted** to this GitHub Pages domain in Google Cloud Console — pasting it elsewhere will fail. If you rotate the key, update both the code and the Cloud Console restrictions.

## Internal docs

Full spec, build plan, and decision history live in the consulting folder (private), not in this repo:
- `_consulting/pricing/quote-tool-spec.md` — design spec
- `_consulting/pricing/BUILD-PLAN.md` — implementation plan + acceptance tests
- `_consulting/DECISIONS.md` — pricing & policy decisions
- `_consulting/ISSUES.md` — open issues
