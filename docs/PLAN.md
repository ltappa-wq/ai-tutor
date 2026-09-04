# Spec-driven plan

Work only against SPEC.md. Each phase ends with something visible in the Floot preview.

## Phase 0 — Grounding (this commit)

- Vision, spec, data model, this plan
- Floot project created: AI Tutor
- Design principles + tokens (calm evening study tool, not toy classroom)

Exit: docs exist; Floot project id recorded below.

**Floot project:** `c8878c21-29d0-49c7-b044-c63b1d29ad38`  
**GitHub:** `ltappa-wq/ai-tutor`

## Phase 1 — UI scaffold (no backend)

Build the student **Today** page with hardcoded sample data:

- Header (student, date, time budget)
- Plan list with statuses
- Start-next CTA
- Session panel or `/session/:itemId` with timer, steps, coach placeholder
- Add-assignment sheet
- Parent status view (`/parent` or a tab)

Exit: preview looks like the product. Clicking around works. Data is fake.

## Phase 2 — Household backend

Provision Floot:

- database
- auth

Schema from DATA-MODEL.md. Seed one household, one parent, two student profiles, courses, assignments, a plan for today.

Endpoints (POST/GET only):

- `GET /api/today` — plan + items for selected student + date
- `POST /api/assignments` — create
- `POST /api/plan/generate` — build/replace remaining items
- `POST /api/sessions/start`
- `POST /api/sessions/event` — step check, stuck, complete
- `GET /api/parent/status`

Exit: Today reads from the DB. Refresh persists.

## Phase 3 — Real tutor loop

Wire `@floot/ai` (or connected model resource) to:

1. Plan generation
2. Step breakdown for an assignment
3. Constrained session coach
4. 5-question prep quiz

Prompt files live as helpers. Refusal rules from SPEC §6 are in the system prompt, not vibes.

Exit: a live session produces steps and a quiz from a real assignment title + notes.

## Phase 4 — Parent close-the-loop

- DailyStatus row written when the student ends the night or last session completes
- Parent page is live data
- Optional email digest via `@floot/email` (off by default)

Exit: parent can answer "did they work?" without opening the chat log.

## Phase 5 — Hardening

- Empty / error / loading states
- Cost cap per household per day on AI calls
- Basic tests on plan-generation helper and session rules
- Publish web preview domain

## Phase 6 — Backlog (not now)

- Student-owned logins
- Photo-of-worksheet / vision homework
- Lecture audio → notes → quiz (Plaud-class pipeline)
- Google Classroom import
- FSD-style "Joe Mode" kid lock + time windows
- Native TestFlight

## Decision log

| Date | Decision |
| --- | --- |
| 2026-09-03 | Family-first household app, not school SaaS |
| 2026-09-03 | Coach-not-cheat is a spec rule, not a slogan |
| 2026-09-03 | v1 student ages 6–12 grades; under-13 is parent-managed |
| 2026-09-03 | Floot is the app runtime; this GitHub repo is the spec repo |

## How we work

1. Change the spec.
2. Change the plan phase notes.
3. Implement only that slice in Floot.
4. Checkpoint in Floot after each phase.
