# Data model — v1

Postgres, snake_case columns. Floot + Kysely. UUIDs for ids. `timestamptz` for times.
Files live in Floot private storage; this schema holds metadata and extracts.
Passwords and LMS tokens are not columns. See AUTH.md.

## household

- id
- name
- timezone (default `America/Chicago`)
- crawl_hour_local int default 14
- created_at

## app_user

- id
- household_id
- role: `parent` | `student`
- display_name
- email nullable  -- parents from Google; students usually null
- google_sub nullable  -- parent SSO subject, unique
- username nullable  -- student login, unique when present
- created_at

Student passwords live in Floot auth. Parent has no app password.

## student_profile

- id
- household_id
- user_id  -- required once the parent has assigned a login
- display_name
- grade_level nullable
- work_block_minutes default 20
- break_minutes default 5
- weekday_time_budget_minutes default 90
- created_at

## lms_connection

Household-level parent portal (v1), not one token per kid.

- id
- household_id
- student_id nullable  -- null when scope is `household`
- scope: `household` | `student`
- provider: `schoology` | `classroom` | `canvas` | `manual`
- portal_role: `parent` | `student` | `district_app`
- status: `connected` | `needs_reauth` | `disabled` | `error`
- external_user_id nullable  -- parent portal user id
- portal_username nullable  -- identifier only
- token_expires_at nullable
- last_crawl_at nullable
- last_error text nullable
- created_at

## lms_child_map

- id
- lms_connection_id
- student_id
- external_child_id
- external_child_name nullable
- unique (lms_connection_id, external_child_id)
- unique (lms_connection_id, student_id)

## course

- id
- household_id
- student_id
- name
- color
- teacher_name nullable
- lms_provider nullable
- lms_section_id nullable
- archived_at nullable

## material_folder

- id
- student_id
- course_id nullable
- slug text
- title
- unique (student_id, course_id)

## material

- id
- household_id
- student_id
- folder_id
- course_id nullable
- source: `upload` | `schoology` | `email` | `photo`
- kind: `notes` | `lecture` | `homework` | `syllabus` | `quiz` | `test` | `rubric` | `slide_deck` | `reading` | `other`
- title
- original_filename
- mime_type
- byte_size int
- storage_key text
- checksum_md5 nullable
- lms_external_id nullable
- lms_path text nullable
- lms_downloaded_at nullable
- summary text nullable
- ingest_status: `pending` | `ready` | `failed`
- ingested_at nullable
- created_at

## extracted_item

- id
- student_id
- course_id nullable
- material_id nullable
- assignment_id nullable
- kind: `homework` | `quiz` | `test` | `reading` | `project` | `note`
- title
- due_on date nullable
- due_at timestamptz nullable
- confidence real nullable
- notes text nullable
- created_at

## course_snapshot

- id
- student_id
- course_id
- current_topic text nullable
- next_assessment_on date nullable
- next_assessment_title text nullable
- open_work_summary text nullable
- planner_hint text nullable
- as_of timestamptz
- unique (student_id, course_id)

## crawl_run

- id
- household_id
- student_id nullable
- provider text
- started_at
- finished_at nullable
- status: `running` | `ok` | `partial` | `failed`
- files_seen int default 0
- files_new int default 0
- assignments_upserted int default 0
- error_summary text nullable

## assignment

- id
- household_id
- student_id
- course_id nullable
- title
- notes text
- type: `homework` | `project` | `quiz` | `test` | `reading` | `other`
- source: `manual` | `schoology` | `extract`
- lms_external_id nullable
- due_on date nullable
- estimate_minutes int nullable
- status: `open` | `done` | `archived`
- created_at
- completed_at nullable

## plan_day

- id
- student_id
- day date
- time_budget_minutes
- generated_at nullable
- generation_notes text nullable
- unique (student_id, day)

## plan_item

- id
- plan_day_id
- assignment_id nullable
- material_id nullable
- kind: `assignment` | `study` | `quiz` | `break` | `admin`
- title
- rationale text nullable
- sort_order int
- planned_minutes int
- status: `pending` | `in_progress` | `done` | `skipped` | `stuck`
- stuck_note text nullable

## work_session

- id
- student_id
- plan_item_id
- started_at
- ended_at nullable
- elapsed_seconds int default 0
- status: `active` | `completed` | `abandoned`

## session_step

- id
- work_session_id
- sort_order
- label
- done boolean default false

## artifact

- id
- student_id
- work_session_id nullable
- assignment_id nullable
- material_id nullable
- kind: `quiz` | `outline` | `critique` | `recap` | `notes` | `worked_solution`
- payload jsonb
- created_at

## daily_status

- id
- student_id
- day date
- items_done int
- items_total int
- minutes_worked int
- stuck_count int
- summary text
- unique (student_id, day)

## ai_log

- id
- household_id
- student_id nullable
- purpose text
- model text
- input_tokens int nullable
- output_tokens int nullable
- created_at

## Indexing (minimum)

- app_user (username) unique where username is not null
- app_user (google_sub) unique where google_sub is not null
- assignment (student_id, status, due_on)
- material (student_id, folder_id, created_at desc)
- extracted_item (student_id, due_on)
- plan_day (student_id, day)
- plan_item (plan_day_id, sort_order)
- work_session (student_id, started_at desc)
- crawl_run (household_id, started_at desc)
