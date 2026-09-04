# 11pm guardian digest

Locked 2026-09-03.

At **23:00 in the household timezone** the app emails **every current parent/guardian** a status report for the household.

This is not optional in v1. Per-address mute is P1. Leaving the household is the off switch.

## Schedule

| Job | Local time | Purpose |
| --- | --- | --- |
| LMS crawl | 14:00 | Pull today's new materials + 7-day dues |
| Guardian digest | 23:00 | What happened today + what is next |

Both use `household.timezone` (default `America/Chicago`). Both fan out with `@floot/email` to every `app_user` with `role=parent` in that household.

Skip the digest only when the household has zero student profiles. Still send if nobody opened the app — “nothing recorded today” is useful.

## One email, all kids

Subject: `AI Tutor — Thursday 9/3 — 2 of 5 done` (household rollup).

Body, per student, in stable name order:

### Today

- Plan items done / total / stuck / skipped
- Minutes in sessions
- Each completed item title
- Each stuck item + the stuck note
- Items never started
- Crawl line if relevant (“3 new Algebra files at 2pm” or “crawl missed — reconnect or upload”)

### Tomorrow

- Hard dues
- Planned study if we already generated tomorrow
- Tests/quizzes extracted for that date

### Next 7 days

A short list, not a novel:

- Date + student + item + kind (homework / quiz / test / project)
- Source tag: LMS / uploaded / manual

End with one link to the parent home (published domain, not the Floot preview).

No full chat transcripts in email. No file attachments. No grades.

## Generation

1. For each student, upsert `daily_status` from sessions + plan items (even if they never tapped “end night”).
2. Read `extracted_item` + `assignment` where `due_on` is tomorrow..today+7.
3. Read latest `crawl_run` for the day.
4. Render one HTML + plaintext email.
5. Send to all guardian addresses.
6. Write `digest_send` so we do not double-send if the job retries.

Idempotent on `(household_id, day)`.

## Quiet hours / weekends

Send every night including weekends. A Saturday “nothing due, nothing worked” mail is cheap and honest. P1 can add “weekdays only.”
