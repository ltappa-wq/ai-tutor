# Spec-driven plan

Work only against the docs. Each phase ends with something visible in the Floot preview.

## Phase 0 — Grounding

Docs + Floot project.

**Floot project:** `c8878c21-29d0-49c7-b044-c63b1d29ad38`  
**GitHub:** `ltappa-wq/ai-tutor`

## Phase 1 — UI scaffold (no backend)

Today + session + parent status + Materials folders. Sample data.

## Phase 2 — Household backend

Database + Google parent SSO + student username/password + guardian invites.

## Phase 3 — Materials ingest (upload first)

Upload, ingest AI, course snapshots, 7-day extracts into the planner.

## Phase 4 — Schoology crawl

Household parent-portal connection + 14:00 job.

## Phase 5 — Real tutor loop

Plan, steps, coach, quiz. Step-then-answer.

## Phase 6 — Parent close-the-loop

- Parent home for every kid
- `daily_status` written from sessions even without an explicit end-night tap
- **23:00 household-tz digest email to every guardian** (DIGEST.md)

Exit: a sample household gets one mail covering today / tomorrow / next 7 days.

## Phase 7 — Hardening

Empty states, cost cap, reconnect, publish.

## Phase 8 — Backlog

Other LMS adapters, photo/voice, lecture audio, digest mute, native stores.

## Decision log

| Date | Decision |
| --- | --- |
| 2026-09-03 | Family-first household app |
| 2026-09-03 | Steps first, then the worked answer |
| 2026-09-03 | Parent Google SSO; kids get parent-assigned username/password |
| 2026-09-03 | Co-parents are equal; invites; all guardians get household email |
| 2026-09-03 | Schoology crawl 14:00 household tz; read-only |
| 2026-09-03 | Repo = per-course folders + General |
| 2026-09-03 | Upload-only is a first-class fallback |
| 2026-09-03 | Guardian digest at 23:00 household tz, every night |

## How we work

1. Change the spec.
2. Change the plan.
3. Implement that slice in Floot.
4. Checkpoint.
