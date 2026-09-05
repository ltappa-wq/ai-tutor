# Spec-driven plan

Work only against the living docs. `GEMINI-*.md` are review artifacts (see REVIEW-GEMINI.md).

## Phase 0 — Grounding

Docs + Floot project.

**Floot project:** `c8878c21-29d0-49c7-b044-c63b1d29ad38`  
**GitHub:** `ltappa-wq/ai-tutor`

## Phase 1 — UI scaffold (no backend)

Today + session + parent status + Materials folders. Sample data. Parent badge mock: Connected / Syncing / Action Required.

## Phase 2 — Household backend

Database + Google parent SSO + student username/password + guardian invites.

## Phase 3 — Materials ingest (upload first)

Upload, ingest AI, course snapshots, 7-day extracts into the planner.

## Phase 4 — Schoology crawl

- Household parent-portal **OAuth / district token** (not password scrape)
- Secrets in Floot secret store
- 14:00 household-tz job, ≤1 req/sec, last 24h files + 7-day dues
- Status machine + parent badge
- Failure is non-blocking: Today, sessions, and 23:00 digest still run; upload fallback stays live
- Reconnect UX when token dies or API returns auth failure

Exit: “Run crawl now” or the 2pm job files at least one material or explains why (reconnect / upload).

## Phase 5 — Real tutor loop

Plan, steps, coach, quiz. Step-then-answer.

## Phase 6 — Parent close-the-loop

Parent home for every kid. `daily_status`. 23:00 digest to every guardian. Digest includes crawl line (ok / missed / action required).

## Phase 7 — Hardening

Empty states, cost cap, reconnect, publish.

## Phase 8 — Backlog

Other LMS adapters, photo/voice, lecture audio, digest mute, native stores. Password-vault + headless crawl only if the board never grants API access *and* we explicitly reopen that decision.

## Decision log

| Date | Decision |
| --- | --- |
| 2026-09-03 | Family-first household app |
| 2026-09-03 | Steps first, then the worked answer |
| 2026-09-03 | Parent Google SSO; kids get parent-assigned username/password |
| 2026-09-03 | Co-parents equal; invites; all guardians get household email |
| 2026-09-03 | Schoology crawl 14:00 household tz; read-only |
| 2026-09-03 | Repo = per-course folders + General |
| 2026-09-03 | Upload-only is a first-class fallback |
| 2026-09-03 | Guardian digest at 23:00 household tz, every night |
| 2026-09-05 | Gemini review: take status/UX/log hygiene; reject password vault + Playwright as v1 |
| 2026-09-05 | Crawl failure never blocks Today or the 11pm digest |
