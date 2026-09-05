# Homework review bench

Locked 2026-09-05.

When the app plans or checks homework, **one model does not get to be judge and jury**.

## Roles

| Role | Job | Sees |
| --- | --- | --- |
| Planner | Break the assignment into steps and a proposed worked solution | Student materials, the prompt, the writing/solver contracts |
| Reviewer | Independently solve or score the same item. Does not see the planner's answer first. | Same materials, same prompt. Separate model. |
| Referee | Only runs if planner and reviewer disagree. Picks a winner or a merged correction and says why in one short note. | Both outputs + the source materials. Third model. |

If planner and reviewer agree, skip the referee. Do not spend three models on `2+2`.

## What the student sees

Unchanged pedagogy:

1. Steps first.
2. Their attempt.
3. Then the **agreed** worked answer (or the referee's pick).

They do not see "Model A vs Model B." Parents do not get a raw model fight in the 11pm mail. Session artifact stores `planner`, `reviewer`, `referee` payloads for later audit (P1 parent transcript).

## When it runs

- Building the session checklist + worked check for homework / quiz / short-answer.
- Writing outline and rubric check (referee only if the critique contradicts the outline).
- Not on every chat turn. Not on "what page is this on?"

## Models

Use three **different** `@floot/ai` models. Do not name vendors in the UI. Swap the trio in one helper (`reviewBench`) without touching session UI.

Log all three calls in `ai_log` with purpose `plan` | `review` | `referee`.

## Failure

- Reviewer down → show planner output, mark `review_skipped`.
- Referee down after a disagreement → show both as "we are not sure; here is the planner path" and flag stuck for the parent digest. Never invent a fake consensus.
