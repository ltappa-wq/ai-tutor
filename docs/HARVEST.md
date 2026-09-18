# Parent-run harvest

Updated 2026-09-17.

Files do **not** come from a silent 2pm Schoology scrape. A parent already logged into Schoology (and PowerSchool) runs a computer-use harvest. Official district APIs stay a future rung.

## Staging tree

```
Google Drive / AI Tutor Harvest /
  Joey / {Course} / …packets
  Joey / Daily Plans / Daily Plan — YYYY-MM-DD
  Reagan / {Course} / …packets
  Reagan / Daily Plans / Daily Plan — YYYY-MM-DD
  _harvest_state.json
```

Course folder names match the LMS section when possible. Do not invent a folder per file.

First run is full. Later runs copy only new ids. `_NOT_COLLECTED_*.md` lists school-domain Drive, Schoology-hosted PDFs, Forms, and blank parent-view agendas — those are not “in Drive.”

## PowerSchool

Grade snapshots land as `PowerSchool_{Class}_YYYY-MM-DD.json` in the matching course folder (or next to Daily Plans). They feed the daily plan (missing, C or lower, %). They are not the assignment files.

## Into the app

Parent imports the student tree on Materials, or a later Drive→app sync. Same ingest + tier caps as upload. See DAILY-PLAN.md for the report the harvest is for.

## Skill

- Grok: `~/.grok/skills/schoology-harvest/`
- Repo: `skills/schoology-harvest/SKILL.md`

The skill does not store a school password.
