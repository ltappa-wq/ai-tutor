# Materials repository + LMS crawl

Locked 2026-09-03. Credential store added 2026-09-03.

The app keeps a **household document repository** per student. An LMS connector (v1: Schoology) crawls new course materials daily. An ingest AI files them, extracts homework / tests / current topics, and feeds the planner for **tomorrow and the next 7 days**.

Manual upload always works. The crawl is how the repo stays current without a parent hunting folders at 7pm.

## Unattended login (what we actually store)

Goal: 2pm job runs with **nobody sitting at a login screen**.

We do that by saving a **long-lived connection** the first time a parent links the student — not by keeping the school password in a table and replaying the website login every afternoon.

### What gets saved

Per student `lms_connection`:

| Saved | Where | Shown in UI |
| --- | --- | --- |
| Provider (`schoology`) + student + section map | Postgres | Yes |
| School username / email (identifier only) | Postgres | Yes |
| OAuth access token + token secret (Schoology OAuth 1.0a) | Floot secret store | Never. Status only: Connected / Needs reconnect |
| Consumer key/secret if the district issued app credentials | Floot secret store | Never |
| School **password** | **Not stored** | — |

The 2pm job reads the secret, calls the API, writes files. No human in the loop until the token dies.

### Why not save the password

- Most districts wrap Schoology in **Clever / ClassLink / Google SSO**. A saved password against schoology.com often cannot log in at all.
- MFA and 90-day token TTL already exist. A stored password still breaks; a stored OAuth token is the thing the API is designed to reuse.
- A plaintext (or even reversible) school password in our DB is the one leak that gets a family wrecked and the app shut off by the district.
- Schoology personal API keys are not a multi-user backdoor. Official path is: user authorizes once, we keep the token.

Connect once. Crawl daily. Reconnect when status flips to `needs_reauth` (~90 days, or if the student changes password / SSO).

### Connect flow (parent, once per student)

1. Settings → this student → Connect Schoology.
2. Enter the **school username** (so we know whose sections to pull). Password is typed into the official OAuth / school login page, not into our form if we can avoid it.
3. App receives token + token secret. Floot secret store keeps them. Postgres keeps `status=connected`, `token_expires_at`, username, last crawl.
4. Immediate test pull: list sections. Parent confirms course folders.
5. Job attached to 14:00 household tz.

“Remember me” in the product means **remember the token**, not remember the password.

### Reconnect

- Banner on Today + Parent status: “Schoology needs a reconnect for Ava — crawl paused.”
- One button. Same flow as first connect. Old secret replaced, not appended.
- Parent can Disconnect: secret deleted, materials already in the repo stay.

### If the district will not grant API access

Upload-only remains first-class. We do **not** add a headless-browser “saved password and scrape the site” mode as v1. That is brittle, often against school AUP, and dies the first time SSO or a captcha appears. If a district later issues a proper app key, we drop it into the same secret slot.

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

Lookback: files / assignments / events whose LMS timestamp is in the last **24 hours**, plus any assignment due in the next **7 days** even if the file is older.

Per connected student, the job:

1. Load the stored token from the secret store. If missing or expired → mark `needs_reauth`, stop.
2. List enrolled Schoology sections → upsert `course` rows.
3. Pull documents, folders, assignments+attachments, tests/quizzes, next-7-day events, last-24h updates with files.
4. For each new file (new external id or new md5): download, store privately, create `material` in the matching folder.
5. Run ingest AI.
6. Upsert `assignment` rows from LMS due dates.
7. Write `course_snapshot`.
8. Write `crawl_run`.
9. Mark the student plan stale.

Idempotent. Nobody is logged into a browser at 2pm. The job is the user.

## Ingest AI

Per file: kind, course guess, topic tags, printed due date, summary, extracted action items.

Per course rollup: current topic, open homework, upcoming quizzes/tests in 7 days, planner hints.

## Planner use

Tonight = Today screen. Extracts also feed tomorrow + days +1..+7.

## Adapter

`LmsAdapter.listSections()`, `listRecentMaterials(since)`, `listUpcomingAssessments(until)`, `downloadFile(id)`.

v1: `SchoologyAdapter` only. Read-only. Never submit.

## Upload path (always on)

PDF, images, DOCX, PPTX, text into General or a course folder. Same ingest AI. This is the product if the district blocks the API.

## Privacy

- Materials household-private. Not used to train public models.
- Under-13: parent owns the connection and the reconnect.
- Secrets never in git, never in client JS, never in `console.log`, never in `crawl_run.error_summary`.
- Crawl logs keep file names + ids, not full document text beyond the short extract.

## Failure behavior

- Crawl fails → keep last good repo, show missed-today + reconnect if auth.
- One file ingest fails → store as `other`, continue batch.
- Auth dead → do not retry in a loop.
