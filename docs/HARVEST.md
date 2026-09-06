# Parent-run harvest (replaces silent Schoology scrape)

Locked 2026-09-05.

The 2pm unattended API crawl is **not** how files get in unless the district later grants an official API. Day to day, a parent who is already logged into Schoology (or Classroom / Canvas) runs the **schoology-harvest** skill in Claude-in-Chrome or a similar computer-use agent.

## Why

District portals sit behind Clever / ClassLink / Google SSO and MFA. Saving a school password and scraping at 2pm is brittle and against the earlier lock. A parent-present browser session is the access we actually have.

## Skill

- Live copy for Grok: `~/.grok/skills/schoology-harvest/`
- Repo copy: `skills/schoology-harvest/SKILL.md`

Run it on demand ("pull Ava's new Schoology files"). First run is full. Later runs use `manifest.json` and skip known ids.

Downloads live in `~/ai-tutor-harvest/{student}/`. The skill does not log in and does not store passwords.

## Into the app

Until we add a folder-watch uploader, the parent (or the skill, talking to the human) drops **new** files on Materials → course folder. Same ingest AI and tier caps as manual upload.

## What the 2pm job still does

- Remind / status only, or no-op if nothing is connected
- Never block Today
- Official API remains a future rung if the board opens it

## Dues without files

`dues.json` from the harvest can seed `extracted_items` / assignments when we wire an import. Until then, the parent can add the assignment on Today.
