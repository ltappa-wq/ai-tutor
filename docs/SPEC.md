# Product spec — v1

Status: draft for first Floot build  
Audience: builder + future-us  
Source of truth with VISION.md, FEATURES.md, AUTH.md, MATERIALS.md, PLAN.md

## 1. Goal

Ship a working household app where:

- A parent signs in with Google, creates the household, and adds students.
- Each student signs in with a username + password the parent assigned.
- A student sees **today's plan**, starts a **guided session**, and marks work done.
- The tutor walks steps first, then may show the worked answer so they can check themselves.
- The tutor never emits a finished take-home essay, lab, or project.
- A parent sees tonight's status for every kid.
- School materials land via parent-portal Schoology crawl when the district allows it, otherwise upload.

## 2. Personas & accounts

Full writeup: AUTH.md.

### Roles

| Role | App login | Can do |
| --- | --- | --- |
| Parent | Google SSO | Own household, add kids, set usernames/passwords, connect Schoology parent portal, view all status, budgets, materials |
| Student | Username + password set by parent | Own Today, sessions, own materials. No LMS connect. No household settings. |

### Auth (v1)

- Parents: Google only (Floot `oauth-login`). First Google login creates the household.
- Extra parents: invite → Google SSO into the same household.
- Students: parent-created username + password (Floot email/password or equivalent hashed credential). No Google on the kid door.
- Student session does not expose a sibling switcher. Parent session can open any kid.
- Under-13 profiles are parent-managed.

### Safety / age

- No public sharing. No open internet browsing inside the tutor.
- Uploads and LMS copies are household-private.
- AI output is tutor-context only. No unrestricted general chat persona.

## 3. Core objects

See DATA-MODEL.md. Summary:

- Household
- App user (parent via Google, student via username)
- StudentProfile
- LMS connection (household parent-portal, mapped to students)
- Course, material folders, materials
- Assignment, extracted items, course snapshots
- PlanDay / PlanItem / Session / Artifact / DailyStatus

## 4. Core loop

```
2:00pm household tz
  → Parent-portal crawl (if connected) files new materials + 7-day dues
After school
  → Student signs in (username / password)
  → Today already has a plan from extracts + assignments
  → Start next → guided session
  → After steps: worked answer for check
  → Done or stuck note
  → Parent Google-SSOs later → status for every kid
```

## 5. Feature requirements

P0/P1/P2/Out catalog lives in FEATURES.md. Materials/crawl in MATERIALS.md. Auth in AUTH.md.

### 5.1 Today (student home)

Must show:

- Student name + date
- Time budget remaining (e.g. 90 minutes)
- Ordered list of plan items with status: pending / in progress / done / stuck
- Primary CTA: **Start next**
- Secondary: Add assignment, regenerate plan

Empty state: "Add tonight's work" — title, subject, due date, estimated minutes.

### 5.2 Daily plan generation

Input: open assignments, extracted items, course snapshots (current topic, next assessment), time budget, block/break lengths.

Output: ordered PlanItems for today; 7-day hints stored for the Week strip.

Rules: regenerating does not delete completed items; student can skip/reorder.

### 5.3 Guided homework session

Steps first, then worked answer. Compressed path if they demand the answer immediately. No paste-ready essay/lab/project.

UI: timer, checklist, scoped chat, I'm stuck, reveal check, Done.

### 5.4 Prepare / study

Quiz from pasted notes or repo materials. Photo-of-notes is P1.

### 5.5 Assignment assist (writing / projects)

Outline, thesis options, paragraph critique, rubric check. Not a complete draft.

### 5.6 Parent home

After Google SSO:

- Per student: done / total, minutes, stuck notes, tomorrow dues, 7-day assessments
- Last crawl status
- Materials browser
- Add kid / reset kid password / connect Schoology

No grade scraping in v1.

### 5.7 Courses & assignments CRUD

Parent or student can add course + assignment. LMS upserts do not overwrite parent edits on the same title without a merge rule (LMS id wins on due date; manual notes stay).

## 6. AI behavior spec

Tutor, then checker. Not ghostwriter. Ground in repo materials when present. Solver contract in FEATURES.md. Log purpose + student + tokens.

## 7. Non-goals (v1)

- Teacher hub
- Community Q&A
- Live human tutors
- Auto-submit to Schoology
- Saving Schoology passwords and scraping the parent website
- Kid Google login
- District SSO as our product (we may *consume* a district API they give us)

## 8. UX constraints

Phone after school. One primary action. Checklists. Calm evening theme. No infinite chatbot as home.

## 9. Analytics (internal)

Household-private: plan generated, session started/completed, stuck, assignment created, quiz completed, worked answer revealed, crawl ok/fail. No ad pixels.

## 10. Success criteria for first Floot preview

1. Today is readable from sample data.
2. Session shows steps + timer + coach.
3. Add assignment works.
4. Parent home shows two kids of status.
5. Login screens exist in the scaffold: Google parent, username student (can be mocked until auth is provisioned).
