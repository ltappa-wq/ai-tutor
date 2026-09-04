# Data model — v1

Postgres, snake_case columns. Floot + Kysely. UUIDs for ids. `timestamptz` for times.
Files live in Floot private storage; this schema holds metadata and extracts.

## household

- id
- name
- timezone (default `America/Chicago`)
- crawl_hour_local int default 14
- created_at

## app_user

- id
- household_id
- email
- role: `parent` | `student`
- display_name
- created_at

## student_profile

- id
- household_id
- user_id nullable
- display_name
- grade_level nullable
- work_block_minutes default 20
- break_minutes default 5
- weekday_time_budget_minutes default 90
- created_at

## lms_connection

- id
- household_id
- student_id
- provider: `schoology` | `classroom` | `canvas` | `manual`
- status: `connected` | `needs_reauth` | `disabled` | `error`
- external_user_id nullable
- token_expires_at nullable
- last_crawl_at nullable
- last_error text nullable
- created_at

Secrets (OAuth tokens, consumer key/secret) are Floot secrets, not columns.

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
- unique (student_id, lms_provider, lms_section_id) where those are not null

## material_folder

- id
- student_id
- course_id nullable  -- null = General
- slug text  -- `general` or course-derived
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
- storage_key text  -- Floot private storage
- checksum_md5 nullable
- lms_external_id nullable
- lms_path text nullable  -- original folder path in Schoology
- lms_downloaded_at nullable
- summary text nullable
- ingest_status: `pending` | `ready` | `failed`
- ingested_at nullable
- created_at
- unique (student_id, lms_provider implied via connection, lms_external_id) where lms_external_id is not null

## extracted_item

Planner fuel pulled from a material or raw LMS assignment object.

- id
- student_id
- course_id nullable
- material_id nullable
- assignment_id nullable  -- linked once upserted
- kind: `homework` | `quiz` | `test` | `reading` | `project` | `note`
- title
- due_on date nullable
- due_at timestamptz nullable
- confidence real nullable
- notes text nullable
- created_at

## course_snapshot

One current picture per course, rewritten after each crawl.

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
- student_id
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
- purpose text  -- plan | quiz | coach | critique | solve | ingest | crawl_rollup
- model text
- input_tokens int nullable
- output_tokens int nullable
- created_at

## Indexing (minimum)

- assignment (student_id, status, due_on)
- material (student_id, folder_id, created_at desc)
- material (checksum_md5) where not null
- extracted_item (student_id, due_on)
- plan_day (student_id, day)
- plan_item (plan_day_id, sort_order)
- work_session (student_id, started_at desc)
- crawl_run (student_id, started_at desc)
