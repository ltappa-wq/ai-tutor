# Plan Update: Phase 4 Schoology Crawl Hardening

Integration notes to update `docs/PLAN.md` with security and resilience guardrails for the Phase 4 crawl.

---

## Updated Phase 4 Milestones

### Phase 4 — Schoology Crawl & Credential Vaulting
* **4.1 Credential Vaulting:**
  * Implement Cloud KMS envelope encryption (`AES-256-GCM` + `GenerateDataKey`).
  * Web API encrypt-only role; isolated worker decrypt-only role.
  * DB migration for `lms_connections`, `lms_sessions`, and `lms_crawl_logs`.
* **4.2 Headless Crawler Engine:**
  * Containerized headless browser (Playwright) running in isolated sandbox.
  * Parent portal authentication with session cookie reuse.
  * Rate-limiting and polite crawl backoff (1 request/sec max).
* **4.3 Ingestion Pipeline:**
  * 14:00 household-tz scheduled dispatch.
  * Extract active course enrollments, upcoming 7-day assignments, and attached PDF/syllabi links.
  * Pipe extracted files directly into the Phase 3 Materials ingest queue.
* **4.4 Reconnect & Error UX:**
  * Parent dashboard status badge (`Connected`, `Action Required`, `Syncing`).
  * Non-blocking failure: If crawl fails, fall back to Phase 3 manual upload without disrupting student sessions or nightly digest.

---

## Decision Log Additions

| Date | Decision |
| :--- | :--- |
| 2026-09-04 | Schoology credentials vaulted using KMS envelope encryption (AES-256-GCM + AAD). |
| 2026-09-04 | Privilege separation: API server is encrypt-only; background crawler worker is decrypt-only. |
| 2026-09-04 | Schoology session cookies cached (encrypted) to minimize raw password decryption cycles. |
| 2026-09-04 | Crawl failures trigger non-blocking 'Reconnect Required' status; upload fallback remains active. |
