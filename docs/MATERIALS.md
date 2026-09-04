# Materials repository + LMS crawl

Locked 2026-09-03.

The app keeps a **household document repository** per student. An LMS connector (v1: Schoology) crawls new course materials daily. An ingest AI files them, extracts homework / tests / current topics, and feeds the planner for **tomorrow and the next 7 days**.

Manual upload always works. The crawl is how the repo stays current without a parent hunting folders at 7pm.

## Repository shape

Per student:

```
Materials
  General/          ← household-wide, not tied to one class
  {Course name}/    ← one folder per course
    2026-09-03-worksheet.pdf
    syllabus.pdf
```

Folder rules:

- One folder per `course` row + exactly one `General` folder.
- Creating a course creates its folder.
- Ingest AI may *propose* a folder; a parent can move a file.
- No deep nested teacher-folder clone. Flat per course is the v1 UX. Original LMS path is stored as metadata.
- Soft-delete only. Crawl re-downloads are deduped, never duplicated.

Allowed kinds (enum `material_kind`):

`notes` | `lecture` | `homework` | `syllabus` | `quiz` | `test` | `rubric` | `slide_deck` | `reading` | `other`

Sources:

`upload` | `schoology` | `email` | `photo`

## Daily crawl

Default schedule: **14:00 in the household timezone** (`America/Chicago` for us). Configurable per household later; 2pm is the v1 default so materials land before the after-school plan generates.

Lookback: files / assignments / events whose LMS timestamp is in the last **24 hours**, plus any assignment due in the next **7 days** even if the file is older (syllabus already in the course, new due date attached).

Per connected student, the job:

1. Refresh OAuth token if needed.
2. List enrolled Schoology sections → upsert `course` rows (name, section id, teacher).
3. Pull:
   - section documents (`GET /sections/{id}/documents`)
   - section folder tree (`/courses/{section_id}/folder/...`)
   - assignments + attachments (`/sections/{id}/assignments?with_attachments=1`)
   - grade items that are tests/quizzes if distinct
   - section events / calendar items in the next 7 days
   - recent updates/discussions with attachments (last 24h)
4. For each new file (new external id or new md5):
   - download via `download_path`
   - store in Floot private storage
   - create `material` row in the matching course folder (or General if unclassified)
5. Run ingest AI on new + updated materials (see below).
6. Upsert `assignment` rows from LMS due dates.
7. Write `course_snapshot` (current topic, next assessment, open work).
8. Write `crawl_run` log (counts, errors, token use).
9. Flag the student’s next plan generation as stale so Today rebuilds off the new extract.

Idempotent. Re-running at 2:05 must not duplicate files.

## Ingest AI

Runs on each new material and on a rollup per course after the crawl.

Per file, extract:

- `material_kind`
- course guess (if the LMS section was missing or “Updates”)
- title / unit / topic tags
- due date if printed on the page
- whether it *is* homework, a study guide, a rubric, or reference
- short summary (2–4 sentences, student-readable)
- extracted action items (`extracted_item`: homework | quiz | test | reading | project | note)

Per course rollup after the batch:

- **Current topic** (what they are covering this week)
- Open homework
- Upcoming quizzes / tests in the next 7 days
- Missing pieces (syllabus never seen, no materials in 14 days)
- Planner hints (“Algebra quiz Friday — 25 min practice Thu”)

Those hints are inputs to plan generation for **tomorrow and days +1..+7**, not a replacement for the Today loop.

## Planner use

Plan generation reads:

- open `assignment` rows (manual + LMS)
- `extracted_item` due in the window
- `course_snapshot.current_topic` for study blocks
- household time budget

Horizon:

- **Tonight / tomorrow** = the Today screen
- **Next 7 days** = a Week strip the parent and student can open (P0 data, P0/P1 UI)

## Schoology connection (reality check)

Schoology API is OAuth 1.0a at `https://api.schoology.com/v1/`. Useful objects exist: sections, documents, assignments, attachments with `download_path` + `timestamp` + `md5_checksum`, course folders, events.

Constraints we do not hand-wave:

- Personal API keys are locked down; they often cannot see other users. A parent app usually needs the **student to authorize** (or a district App Center app).
- Tokens expire (~90 days). Reconnect UI is required.
- Rate limit: ~50 API credits / 5 seconds. GET costs 1. A two-kid, six-section crawl must be polite and paged.
- Districts can block third-party apps. **Upload-only mode is a first-class fallback**, not an error state.
- We never write back to Schoology in v1. No submissions. Read + copy files out.

Connect flow (parent):

1. Settings → Connect Schoology for a student.
2. OAuth (or paste consumer key/secret if the district issued one — last resort, stored as a Floot secret, never in the repo).
3. Test pull: list sections. Map each section to a course folder.
4. Run first crawl immediately, then attach to the 2pm schedule.

Adapter interface so Classroom/Canvas can plug in later without rewriting the repo:

`LmsAdapter.listSections()`, `listRecentMaterials(since)`, `listUpcomingAssessments(until)`, `downloadFile(id)`.

v1 adapter: `SchoologyAdapter` only.

## Upload path (always on)

Parent or student can drop files into General or a course folder: PDF, images, DOCX, PPTX, plain text. Same ingest AI as the crawl. This is how we run the product if the district blocks the API, and how lecture notes that never hit Schoology get in.

## Privacy

- Materials are household-private.
- Not used to train public models.
- Under-13 profiles: parent owns the connection.
- Crawl logs retain file names + ids, not full document text, beyond a short extract.

## Failure behavior

- Crawl fails → keep last good repo, surface “Schoology crawl missed today — upload or retry” on Today and Parent status.
- Ingest AI fails on one file → file still stored as `other`, rest of batch continues.
- Token dead → stop calling the API, do not loop 401s.
