# Spec-driven plan

Work only against SPEC.md + FEATURES.md + MATERIALS.md. Each phase ends with something visible in the Floot preview.

## Phase 0 — Grounding

- Vision, spec, feature map, materials spec, data model, this plan
- Floot project created: AI Tutor

Exit: docs exist.

**Floot project:** `c8878c21-29d0-49c7-b044-c63b1d29ad38`  
**GitHub:** `ltappa-wq/ai-tutor`

## Phase 1 — UI scaffold (no backend)

Today + session + parent status + a Materials screen with General + per-course folders and sample files.

Exit: preview looks like the product. Data is fake.

## Phase 2 — Household backend

Database + auth. Schema from DATA-MODEL.md. Seed household, two students, courses, folders, sample materials, assignments, today's plan.

Exit: Today and Materials read from the DB.

## Phase 3 — Materials ingest (upload first)

- Upload into a folder
- Ingest AI: kind, summary, extracted items
- Course snapshot rewrite
- Plan generation reads extracts for tomorrow + 7 days

Exit: drop a PDF in Algebra, see a homework extract and a planner hint without Schoology.

## Phase 4 — Schoology crawl

- Connect Schoology for a student
- Adapter + 14:00 household-tz scheduled job
- 24h new files + 7-day dues
- Dedup, store, ingest, snapshots, crawl_run log
- Upload-only fallback if token dies

Exit: a 2pm run (or "Run crawl now") brings in a real or stubbed section file.

## Phase 5 — Real tutor loop

`@floot/ai` for plan, steps, coach, quiz grounded in repo materials. Step-then-answer contract.

## Phase 6 — Parent close-the-loop

DailyStatus + crawl status + upcoming assessments on the parent page.

## Phase 7 — Hardening

Empty/error states, AI cost cap, reconnect Schoology, publish.

## Phase 8 — Backlog

- Other LMS adapters
- Photo worksheet, voice
- Lecture audio pipeline
- Student-owned logins
- Native stores

## Decision log

| Date | Decision |
| --- | --- |
| 2026-09-03 | Family-first household app, not school SaaS |
| 2026-09-03 | Steps first, then show the worked answer |
| 2026-09-03 | Grades 6–12; under-13 parent-managed |
| 2026-09-03 | Floot runtime; GitHub is the spec repo |
| 2026-09-03 | Schoology daily crawl at 14:00 household tz |
| 2026-09-03 | Repo = per-course folders + General |
| 2026-09-03 | Ingest AI feeds tomorrow + 7-day plan |
| 2026-09-03 | LMS is read-only. Never submit back. |
| 2026-09-03 | Upload-only is a first-class fallback |

## How we work

1. Change the spec.
2. Change the plan phase notes.
3. Implement only that slice in Floot.
4. Checkpoint in Floot after each phase.
