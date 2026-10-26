# Reverse API Engineer — Test Findings

**Scope:** 7 targets across four difficulty classes, one configuration, same environment. Objective: validate behavior beyond the happy path.

## Results

| Target type | Outcome | Evidence |
|---|---|---|
| Public REST API | **PASS** | Working client; 108 records returned on independent execution |
| SPA + public JSON API | **PASS** | Correct endpoint and query; 548k hits searched live |
| JS-rendered page, no API | **FAIL** | Generated a client for a different site; pagination crashed |
| Social site — bot + auth wall | **FAIL** | Correct endpoint discovered, but replay returned HTTP 403 |
| Heavy SPA — internal API | **PARTIAL** | Endpoints and parser correct; live calls returned errors |
| E-commerce search — anti-bot | **FAIL** | Scraper built, returned 0 results; run timed out |
| Community site — auth + CAPTCHA | **ABORTED** | CAPTCHA solve timed out; model rate-limited |

**Verified working: 2 / 7.** Known spend: $4.08. Total wall time: ~41 minutes.

## What held up

- **Public JSON APIs.** Endpoint discovery, request reconstruction, and working client generation in ~2 minutes.
- **API shape mapping.** Internal endpoints and payload structures were reconstructed correctly even where replay failed.
- **Capture artifacts.** HAR files with 180–208 entries on hard targets, usable for offline analysis.

## What failed

- **Verification gap (highest impact).** `status: "ok"` confirms a client was generated, not that it works. Three runs reported success with broken output.
- **Target drift.** When the requested site had no API, the model substituted an unrelated site instead of scraping the rendered page.
- **Bot detection.** Captured requests replayed outside the browser returned 403 with bot flags in the response.
- **Auth walls and CAPTCHA.** Logged-out sessions captured only a sign-in shell; challenge solving timed out.
- **Stale client state.** Internal API constants (client version, visitor/session data) were frozen at capture time and rejected at runtime.
- **HTML structure drift.** Anti-bot responses and changed selectors produced zero rows.
- **Secret leakage.** Captured session cookies were hardcoded into generated source; some sessions were invented.
- **Operational friction.** Browser-profile collisions between MCPs, sandbox file-access errors, missing HAR paths with non-default drivers, buffered logs.
- **Free-tier rate limits.** Mid-run throttling caused retries, timeouts, and engine failures.

## Assessment

Reliable for open, unauthenticated JSON APIs. Not reliable where correctness depends on bot evasion, authentication, CAPTCHA, rotated client state, or HTML structure — and it does not self-verify those cases. Generated output should always be executed and validated before use.

---

*Full evidence: [reverse-api-engineer.html](reverse-api-engineer.html)*
