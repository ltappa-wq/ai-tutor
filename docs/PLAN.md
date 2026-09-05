# Spec-driven plan

Work only against the living docs. `GEMINI-*.md` are review artifacts (see REVIEW-GEMINI.md).

## Phase 0 — Grounding

Docs + Floot project.

**Floot project:** `c8878c21-29d0-49c7-b044-c63b1d29ad38`  
**GitHub:** `ltappa-wq/ai-tutor`

## Phase 1 — UI scaffold (no backend)

Done. Today + session + parent + Materials with sample data.

## Phase 2 — Household backend

Done enough to log in. Remaining: guardian invite email flow.

## Phase 3 — Materials ingest (upload first)

Upload into folders. Enforce **tier file + storage caps** (TIERS.md). Ingest AI. Course snapshots. 7-day extracts.

## Phase 4 — Schoology crawl

OAuth / district token. 14:00 job. Same caps as upload. Non-blocking failure.

## Phase 5 — Real tutor loop

Plan, steps, coach, quiz. Step-then-answer. **Review bench:** planner model, reviewer model, referee only on disagreement (REVIEW-BENCH.md).

## Phase 6 — Parent close-the-loop

Parent home. 23:00 digest to every guardian.

## Phase 7 — Hardening

Empty states, AI cost cap, reconnect, publish for our household.

## Phase 8 — Distribute

Public Free / Basic / Pro. Checkout later. Limits already enforced from Phase 3. Then: other LMS adapters, photo/voice, lecture audio, digest mute, native stores.

## Decision log

| Date | Decision |
| --- | --- |
| 2026-09-03 | Family-first household app |
| 2026-09-03 | Steps first, then the worked answer |
| 2026-09-03 | Kids get parent-assigned username/password |
| 2026-09-03 | Co-parents equal; invites; all guardians get household email |
| 2026-09-03 | Schoology crawl 14:00 household tz; read-only |
| 2026-09-03 | Repo = per-course folders + General |
| 2026-09-03 | Upload-only is a first-class fallback |
| 2026-09-03 | Guardian digest at 23:00 household tz, every night |
| 2026-09-05 | Gemini review: take status/UX; reject password vault + Playwright as v1 |
| 2026-09-05 | Parents sign in with Google SSO or email + password |
| 2026-09-05 | Storage caps: 25 MB manual upload, 100 MB platform max per file |
| 2026-09-05 | Three-model review bench on homework |
| 2026-09-05 | Public tiers Free / Basic / Pro, gated by files + storage |
