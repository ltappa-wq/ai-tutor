# Data model — v1

Postgres, snake_case columns. Floot + Kysely. UUIDs for ids. `timestamptz` for times.

## household

- id
- name
- timezone (default `America/Chicago`)
- created_at

## app_user

Floot auth user mirror / link.

- id
- household_id
- email
- role: `parent` | `student`  (enum)
- display_name
- created_at

## student_profile

- id
- household_id
- user_id nullable (null if profile-only in v1)
- display_name
- grade_level nullable
- work_block_minutes default 20
- break_minutes default 5
- weekday_time_budget_minutes default 90
- created_at

## course

- id
- household_id
- student_id
- name
- color
- teacher_name nullable
- archived_at nullable

## assignment

- id
- household_id
- student_id
- course_id nullable
- title
- notes text
- type: `homework` | `project` | `quiz` | `test` | `reading` | `other`
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
- kind: `quiz` | `outline` | `critique` | `recap` | `notes`
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

- assignment (student_id, status, due_on)
- plan_day (student_id, day)
- plan_item (plan_day_id, sort_order)
- work_session (student_id, started_at desc)
