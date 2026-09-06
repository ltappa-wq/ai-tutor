---
name: schoology-harvest
description: Harvest course files from a school LMS the parent is already logged into (Schoology first, also Google Classroom or Canvas). Use when the user says pull Schoology docs, run the materials crawl, Claude in Chrome harvest, download new assignments, or sync school files into AI Tutor. Incremental only after the first run. Never ask for or store a school password.
metadata:
  type: workflow
  version: "1.0"
  product: ai-tutor
---

# Schoology harvest (parent-run)

The parent is already logged into the school site in the browser. You operate that session. You do not log in for them. You do not ask for the school password.

Target product is AI Tutor (`ltappa-wq/ai-tutor`). Files land in the household Materials repo via the app upload path after you download them locally.

## When to run

- First run for a student — pull **all** visible course files.
- Later runs (daily after school is the habit) — pull **only files not already in the manifest**.

Ask which student if more than one child is in the parent portal.

## Manifest (source of truth for "new")

Keep one manifest per student on disk, next to the downloads:

`~/ai-tutor-harvest/{student-slug}/manifest.json`

A file is **new** if any of these is true:

- `id` not in the manifest
- `updatedAt` is newer than the stored row
- byte size or sha256 changed

Do not download a file whose id and updatedAt already match.

## Browser procedure (Claude in Chrome, or any computer-use agent)

1. Confirm the parent is logged in. If you see a login wall, stop and tell them to sign in, then rerun the skill. Never type credentials.
2. Open the parent portal and pick the named student.
3. Walk **each course** listed for that student.
4. In each course, open Materials / Course Content / Resources / Assignments.
5. For every downloadable item, record id/title/course/url/updatedAt. Download only if new into `~/ai-tutor-harvest/{student-slug}/{Course}/`.
6. Capture assignment titles and due dates even without a file into `dues.json`.
7. Rewrite `manifest.json` after a successful walk.

## Rate and manners

Pause between courses. Do not submit work. Do not leave the school domain.

## After download

Upload **new** files only into AI Tutor → Materials → that student → matching course folder.

## Report

student, courses walked, files seen/new/skipped, dues in 7 days, errors. Then stop.
