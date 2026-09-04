# Product spec — v1

Status: draft for first Floot build  
Audience: builder + future-us  
Source of truth with VISION.md, FEATURES.md, and PLAN.md

## 1. Goal

Ship a working household app where:

- A parent creates the household and student profiles.
- A student sees **today's plan**, starts a **guided session**, and marks work done.
- The tutor walks steps first, then may show the worked answer so they can check themselves.
- The tutor never emits a finished take-home essay, lab, or project.
- A parent sees tonight's status.

## 2. Personas & accounts

### Roles

| Role | Can do |
| --- | --- |
| Parent | Create household, add students, set subjects/schedule, view status, set daily time budget |
| Student | View own plan, run sessions, capture notes/assignments, mark complete |

### Auth (v1)

- Email/password (Floot auth).
- One login per person. Parent account owns the household.
- **Decision for first slice:** parent login + switchable student profiles on the same device. Separate student logins are P2.

### Safety / age

- Treat under-13 profiles as parent-managed. No public sharing. No open internet browsing inside the tutor.
- Uploads are household-private.
- AI output is filtered for tutor context (schoolwork). No unrestricted general chat persona.

## 3. Core objects

See DATA-MODEL.md. Summary:

- Household
- User (parent)
- StudentProfile
- Subject / Course
- Assignment (title, due, type, status, notes, rubric optional)
- PlanDay (date + ordered items)
- PlanItem (assignment, study block, break, quiz)
- Session (guided work against a PlanItem)
- Artifact (quiz, outline, checklist progress, recap, worked solution)
- DailyStatus (parent digest)

## 4. Core loop

```
Evening / after school
  → Student opens Today
  → Plan already generated (or generate now)
  → Tap first incomplete item
  → Guided session (timer + steps + tutor chat constrained to the item)
  → After steps: show worked answer for check
  → Mark done or park with a "stuck" note
  → Next item
  → End of night → Daily status for parent
```

## 5. Feature requirements

P0/P1/P2/Out catalog lives in FEATURES.md. This section is the P0 behavior spec.

### 5.1 Today (student home)

Must show:

- Student name + date
- Time budget remaining (e.g. 90 minutes)
- Ordered list of plan items with status: pending / in progress / done / stuck
- Primary CTA: **Start next**
- Secondary: Add assignment, regenerate plan, switch student (parent device)

Empty state: "Add tonight's work" — title, subject, due date, estimated minutes.

### 5.2 Daily plan generation

Input:

- Open assignments + due dates
- Optional class schedule / test dates
- Time budget
- Student preferences: work block length (default 20 min), break length (default 5)

Output:

- Ordered PlanItems that fit the budget
- Hard-due items first, then study, then stretch
- Explicit breaks
- Human-readable rationale line per item ("Due tomorrow; 20 min draft outline")

Rules:

- Regenerating a plan does not delete completed items from today.
- Student can drag-reorder or skip.
- Plan is a suggestion with an owner, not a prison.

### 5.3 Guided homework session

For an assignment item:

1. Restate the goal in one sentence.
2. Break into 3–7 checkable steps.
3. Work one step at a time. Check the student's move before advancing.
4. Tutor chat is scoped to this assignment.
5. After the path is walked, show the fully worked solution and the final answer so they can check a worksheet.
6. If they demand the answer on message one: compress the steps (do not run a 20-turn purity ritual at 9:40pm), then show the worked answer.
7. Offer one sibling problem of the same type.

Still forbidden: a paste-ready completed essay, lab report, or multi-page project.

Session UI:

- Timer (default block length)
- Checklist
- Chat
- "I'm stuck" → captures a note for the parent digest
- Reveal check / worked answer
- Done / Keep going

### 5.4 Prepare / study

For a study or test-prep item:

- Student pastes notes or describes the topic (photo-of-notes is P1).
- Tutor produces: 5-question check, then weak-spot recap.
- Score stored as an Artifact.

### 5.5 Assignment assist (writing / projects)

Allowed:

- Outline from prompt + rubric
- Thesis options (student picks)
- Paragraph-level critique (clarity, evidence, grammar flags)
- Rubric self-check

Not allowed:

- Full essay / lab report / take-home project they can put their name on

### 5.6 Parent status

A single page / section:

- Per student, for today:
  - items done / total
  - minutes worked (from session timers)
  - stuck notes
  - tomorrow's hard dues

No grade scraping in v1. Session transcript open is P1.

### 5.7 Courses & assignments CRUD

Parent or student can:

- Add course (name, color, teacher optional)
- Add assignment (title, course, due, type: homework | project | quiz | test | reading, estimate minutes, notes)
- Edit / complete / archive

## 6. AI behavior spec

System posture:

- Tutor, then checker. Not ghostwriter.
- Age-appropriate, calm, specific.
- Prefer questions and next steps over lectures — until the path is done, then show the checkable answer.
- Cite the student's own materials when present.
- If materials are missing, ask for them before inventing content.
- Math / short-answer: solver contract in FEATURES.md.
- Writing: coach structure and evidence; keep the student's voice.

Every AI feature must log:

- purpose (plan | quiz | coach | critique | solve)
- student_id
- plan_item_id if any
- model + token usage (for cost control)

## 7. Non-goals (v1)

See FEATURES.md Out + P2. Headline nos:

- Teacher hub / lesson plans / IEP tools
- Community Q&A and leaderboards
- Live human tutor marketplace
- Google Classroom / Canvas sync
- Lecture-audio pipeline
- District SSO
- Auto-submit to a school portal

## 8. UX constraints

- Fast on a phone after school. Thumb-first.
- One primary action per screen.
- Checklists over walls of text.
- Dark-capable; default a calm evening light theme that does not look like a toy or a bank.
- No infinite chatbot as the home screen.

## 9. Analytics (internal)

Track, household-private:

- plan generated
- session started / completed
- stuck marked
- assignment created
- quiz completed
- worked answer revealed

No third-party ad pixels.

## 10. Success criteria for first Floot preview

A reviewer can:

1. Open Today and understand the night's work from sample data.
2. Start a session and see steps + timer + coach panel.
3. Add an assignment and see it appear.
4. View a parent status summary.

Persistence and real AI can land immediately after that scaffold is honest.
