# Reverse API Engineer — R&D Summary

**What it is:** a Python CLI that turns a website into a typed API client by driving a real browser, recording network traffic to HAR, and letting an LLM generate the client.

## How it works

1. You give it a goal and a starting URL.
2. It spawns a configured agent SDK/model (Claude, OpenCode, Cursor, or Copilot).
3. The agent drives a browser through an MCP server or CLI.
4. The session is recorded to a HAR file.
5. The model reads the HAR and writes `api_client.py` + `README.md`, synced to a local folder.

It supports agent/manual/engineer/collector modes and a scriptable contract (`--json`, `--dry-run`, `--no-interactive`, exit codes 0/1/2).

## Test result (7 targets, 2 verified working)

| Target type | Outcome | What happened |
|---|---|---|
| Public REST API | **PASS** | Working client in ~2 min |
| SPA + public JSON API | **PASS** | Correct endpoint and query; live results verified |
| JS-rendered, no API | **FAIL** | Drifted to an unrelated site; pagination crashed |
| Social — bot + auth wall | **FAIL** | Correct endpoint discovered, replay returned HTTP 403 |
| Heavy SPA — internal API | **PARTIAL** | Endpoints found, but live calls returned errors |
| E-commerce — anti-bot | **FAIL** | Scraper built, returned 0 results, run timed out |
| Community — auth + CAPTCHA | **ABORTED** | CAPTCHA solve timed out; model rate-limited |

## Where it fails

- **False-positive success.** `status: "ok"` means "a client was generated", not "the client works". Three runs reported success with broken output.
- **Target drift.** When a site has no API, the model can substitute a different site instead of scraping the rendered page.
- **Bot detection.** Captured requests replayed outside the browser get 403 (Pinterest).
- **Auth walls and CAPTCHAs.** Logged-out sessions capture only a sign-in shell; solve attempts time out.
- **Stale client state.** Internal APIs (e.g. YouTube Innertube) rotate client versions and visitor data; capture-time constants decay.
- **HTML structure drift.** Anti-bot pages or changed selectors yield zero rows.
- **Secret leakage.** Captured session cookies are hardcoded into generated source; some sessions are invented.
- **Tooling friction.** Browser-profile collisions, MCP file-access sandbox errors, missing HAR paths with non-default drivers, and buffered logs.
- **Free-model rate limits.** Mid-run throttling causes retries, timeouts, and engine failures.

## Bottom line

Useful as a **reconnaissance and scaffolding tool** for open, unauthenticated JSON APIs. Not reliable for protected or stateful targets, and never a substitute for executing and validating the generated client. Known spend across the seven runs: **$4.08**, total wall time **~41 minutes**.

---

*Full analysis: [reverse-api-engineer.html](reverse-api-engineer.html)*
