# Daily plan report

Locked 2026-09-17.

The front door of the product is a **dated plan per child**: what is not turned in, what scored C or lower, what test is this school week, and the one next action to get it done.

Weekends are omitted unless something is actually due that day.

## Live copies (until the app writes them)

Google Drive, not this repo:

```
AI Tutor Harvest /
  Joey / Daily Plans / Daily Plan — YYYY-MM-DD
  Reagan / Daily Plans / Daily Plan — YYYY-MM-DD
```

Folder ids:

- Joey Daily Plans: `1SL0BbEVG54ReB9rCel5_qDDoagW-N2jg`
- Reagan Daily Plans: `12rHpRFAIKFB1IeaD0Fg1cSvzwrd8BEVH`

Each new day is a **new Google Doc**. Do not overwrite yesterday.

## Sections (required, in this order)

1. **Do first** — 1–3 bullets. Highest leverage (missing heavy points, test tomorrow, D/F class).
2. **Not turned in** — table: Due | Class | Assignment | Status.
   Include missing, incomplete, absent-zero, and `--` with a past due date.
3. **Turned in — C or lower** — bullets. ≤76% or letter C / D / F / I. Skip B- and above.
4. **Tests / quizzes this week** — school days through Friday only.
5. **Also** — class % if D/F, category weights that change priority, one stale-data note.

## Inputs

| Source | What it is for |
| --- | --- |
| PowerSchool guardian harvest (JSON per class) | Missing / scores / current % |
| Schoology parent harvest | Packets, weekly agendas, dated quizzes the gradebook has not posted yet |
| MFHS odd/even calendar | 2026-09-10 was even; school days flip after that |

PowerSchool is source of truth for submitted vs missing. Packets are how the student does the work.

## Assist

The report names the work. The session (steps, timer, I’m stuck, worked answer after steps) is how they do it. One Start per “do first” item, packet from that course folder.

## Stale data

Every doc states the harvest date. If PowerSchool is older than tonight, say so at the top.
