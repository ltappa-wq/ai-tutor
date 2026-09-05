# Gemini plan review — what we took

Reviewed 2026-09-05 against `GEMINI-AUTH.md`, `GEMINI-DATA-MODEL-CREDENTIALS.md`, `GEMINI-PLAN-UPDATE.md`.

Gemini designed a **password vault + Playwright parent-portal scraper** with KMS envelope encryption. That is a serious security writeup for a different architecture than the one we locked.

## Take (fold into living docs)

| Item | Where it lands |
| --- | --- |
| Connection status machine: `pending` / `connected` / `syncing` / `invalid_credentials` / `reconnect_required` / `paused` / `error` | DATA-MODEL `lms_connection.status` |
| Parent badge: Connected / Syncing / Action Required | MATERIALS + SPEC parent home |
| Non-blocking crawl failure; student Today and 11pm digest still run | MATERIALS, DIGEST, PLAN Phase 4 |
| Masked account identifier in UI (`j***n@gmail.com`) | DATA-MODEL `portal_username_masked` |
| `portal_domain`, `last_success_at`, `consecutive_failures`, `last_error_code` | DATA-MODEL |
| Crawl log counts: courses / assignments / materials / duration | DATA-MODEL `crawl_run` |
| Polite crawl: ≤1 req/sec, no tight retry on CAPTCHA/2FA | MATERIALS |
| Never log password, token, secret, payload | AUTH + MATERIALS |
| Student cannot see LMS secrets, billing, digest settings | AUTH (already true; restated) |
| Guardian reset of student password from parent home | AUTH (already true) |

## Reject (do not become v1)

| Gemini proposal | Why |
| --- | --- |
| Store Schoology **password** in envelope-encrypted payload | We locked: OAuth/token or district API. Password+scrape dies on SSO/MFA/CAPTCHA and violates many district AUPs. |
| Headless Playwright crawler as Phase 4 | Not a Floot v1 runtime. Brittle. Same ToS problem. |
| Cloud KMS KEK/DEK, encrypt-only API role, decrypt-only worker | Correct pattern *if* we stored passwords. We store Floot-managed OAuth secrets instead. Floot does not give us AWS IAM split. |
| `lms_sessions` cookie-jar cache for scraped logins | Only needed for the scrape design. |
| Decision log line claiming KMS vaulting is locked | It is not. |

## Defer (not no forever)

- If the board never grants API/OAuth and a parent still wants unattended crawl, revisit a **narrow** credential vault then — as a later phase with a threat model, not as the default path.
- Student numeric PIN instead of a passphrase: allowed as an *option* when the parent sets the kid login; not required.
- Per-guardian digest mute (already P1).

## Standing rule

`GEMINI-*.md` files are review artifacts. Living spec is AUTH / MATERIALS / DATA-MODEL / PLAN / DIGEST / SPEC. If they conflict, living spec wins.
