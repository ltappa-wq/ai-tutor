# Feature map

Compiled 2026-09-03 from the Khanmigo vs Brainly review plus ChatGPT Study Mode, Gemini Guided Learning / NotebookLM, Photomath / Mathway / Symbolab, Quizlet / Knowt, Chegg, Socratic/Lens, MagicSchool.
Updated 2026-09-03 with materials repo + Schoology crawl.

Policy lock: **walk the steps first. After the student has worked the path, showing the actual answer is allowed.** We are not Khanmigo-strict. We still do not ghostwrite a submit-ready essay, lab report, or take-home project.

## What the market actually sells

| Camp | Examples | They own | They miss |
| --- | --- | --- | --- |
| Socratic tutor | Khanmigo, ChatGPT Study Mode, Gemini Guided Learning | Hints, checks, no-cheat posture | Tonight's plan, time budget, parent close-loop |
| Answer + steps | Brainly, Photomath, Mathway, Chegg, Symbolab | Camera, worked solution, textbook keys | Sequencing the night, ADHD pacing, household |
| Notes → study objects | Quizlet Magic Notes, Knowt, NotebookLM, StudyFetch | Flashcards, quizzes from *their* materials | Doing the homework sitting in front of them |
| Teacher copilot | Khanmigo Teachers, MagicSchool | Lesson plans, rubrics, IEP drafts | Irrelevant to a family v1 |
| Human backup | Brainly Tutor, Chegg | Live STEM chat | Cost, scheduling, not our v1 |

Nobody good owns: **LMS crawl → course-folder repo → extract dues/topics → after-school plan → timed session → parent status.** That is our product.

## Pedagogy we steal

From Khanmigo / Study Mode / Guided Learning:
- One problem at a time
- Diagnose the stuck point before lecturing
- Hint → student attempt → check → next step
- Knowledge check at the end of a block
- Writing coach that edits, does not author

From Brainly / Photomath:
- After the path is walked, show the worked answer so they can check themselves
- Simplify / expand the same explanation
- Follow-up on a single line ("why divide here?")
- Photo of the worksheet as an input (v1.1)
- Multiple methods when the teacher wants a specific approach

From Quizlet / NotebookLM:
- Turn *their* notes into a 5-question quiz and a recap
- Ground answers in uploaded material when it exists
- Do not invent a chapter they did not give us

## Include list

Priority: P0 ships in first working product. P1 is next slice. P2 is backlog. Out is a hard no for this app.

### P0 — the school-night loop + materials brain

| Feature | Why |
| --- | --- |
| Household + student profiles | Family is the unit |
| Courses + assignments CRUD | Plan has nothing to chew without this |
| Document repository (per-course folders + General) | Source of truth for what school actually assigned |
| Manual file upload into those folders | Works even when the district blocks APIs |
| Schoology connector + daily 2pm crawl (last 24h + 7-day dues) | Stops the 7pm folder hunt |
| Ingest AI: sort, summarize, pull homework / tests / current topic | Turns files into planner fuel |
| 7-day lookahead from extracts | Tomorrow is not enough; Friday's quiz needs Thursday |
| Daily plan generator | Time budget, due dates, block length, breaks |
| Today screen with one Start-next CTA | ADHD / "where do I start" |
| Guided session: timer + checklist + scoped chat | Core tutor |
| Step-then-answer solver | Your rule: finish the path, then reveal the worked answer |
| I'm-stuck note | Feeds the parent digest |
| Parent tonight-status | Done / left / minutes / stuck |
| Writing: outline + rubric check + critique | Not a finished paper |
| Prep quiz from pasted notes *or* repo materials | Grounded in their course |

### P1 — makes it sticky

| Feature | Why |
| --- | --- |
| Photo of worksheet / handwritten problem | Brainly/Photomath table stakes on a phone |
| Simplify / Expand on any explanation | Same answer, two reading levels |
| Worked-example sibling problem | Practice without copying tonight's exact item |
| Flashcards from notes or missed quiz items | Retention after the session |
| Voice in / read-aloud | ADHD, dysgraphia, kitchen-table noise |
| Alternate method for the same problem | "Teacher wants slope-intercept, not point-slope" |
| Session history the parent can open | Transparency without hovering live |
| Daily time windows | Work happens after practice, not at 11:40 |
| Week strip UI for the 7-day extract | Data is P0; nicer calendar can follow |
| Email parent a 2pm crawl summary | "3 new files, quiz Friday" |

### P2 — later, not never

| Feature | Why wait |
| --- | --- |
| Google Classroom / Canvas adapters | Same interface, different district |
| Lecture audio → notes → quiz | Real pipeline, not a weekend |
| Student-owned login | Profile-switch is enough at first |
| Graph / sketch of the math | Nice; Photomath already spent a decade on it |
| Spaced-repetition schedule across days | Needs history first |
| Code sandbox + debug coach | Extra runtime; not the household pain |
| Native store apps | Floot web + PWA first |

### Out — do not build

| Feature | Why |
| --- | --- |
| Teacher hub, lesson plans, IEP writer, report-card comments | Wrong customer |
| Literary character roleplay, debate partner, curiosity games | Not the school night |
| College admissions essay + FAFSA navigator | Different product |
| Community Q&A, points, leaderboards | We are not a forum |
| Live human tutor marketplace | Support nightmare + margin |
| Full essay / lab / take-home project generator | Integrity line we keep |
| Auto-submit to Schoology / any school portal | Read-only crawl. Never write back. |
| District SSO / SIS as a product | Enterprise later, if ever |
| Ad-supported answer feed | We are not Brainly Basic |

## Solver contract (math and short-answer)

1. Restate the problem in one line.
2. Ask what they have already tried, if anything.
3. Name the first move. Wait.
4. Check their step. Correct the move, do not skip ahead.
5. Repeat until the last step.
6. **Then** show the fully worked solution and the final answer so they can check a worksheet.
7. Offer one sibling problem of the same type.

If they demand the answer on message one: still do steps 1–5 in compressed form, then step 6. Tight deadline is a real household constraint.

## Writing contract

Allowed: thesis options they pick, outline, paragraph critique, grammar flags, rubric self-check, "what's missing."
Forbidden: a complete draft they can put their name on.

## Input methods, by phase

| Phase | Inputs |
| --- | --- |
| P0 | Type / paste, file upload, Schoology crawl |
| P1 | Photo of worksheet or board, voice |
| P2 | Lecture audio, other LMS adapters |

## Parent surface (keep small)

Show:
- items done / total
- minutes in sessions
- stuck notes
- tomorrow's hard dues + 7-day assessments from the crawl
- last crawl status (ok / missed / reconnect Schoology)
- optional: open a session transcript (P1)

## Naming in the app

Prefer **Plan / Session / Check / Status / Materials.**
