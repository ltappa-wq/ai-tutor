# Auth and household setup

Locked 2026-09-03. Co-parents locked 2026-09-03.

Two different logins. Do not mix them up.

1. **This app** — how a parent or kid signs into AI Tutor.
2. **Schoology** — how the 2pm crawler reads school materials.

## App login

### Parents / guardians

- Sign in with **Google SSO** only (Floot `oauth-login`).
- First Google sign-in creates the household. That person is `creator` only for bookkeeping. **Access is equal.** There is no admin-only screen the second parent cannot open.
- Add a second or third guardian: any current parent enters their email → we send an invite link → they Google-SSO with that mailbox → they join the household with the **same permissions** as the first parent.
- Cap: 5 guardians per household (enough for two homes + a grandparent; not a staff directory).
- No parent username/password in this app. Google is the parent door.
- A guardian can leave the household. They cannot delete the household if they are the only remaining parent — transfer or add someone first. Any remaining parent can remove another parent (don't be cute with this; log it).

Every parent can:

- Add / archive students
- Assign or reset a student username + password
- Invite / remove other guardians
- Connect or reconnect Schoology
- Map Schoology children to profiles
- See status and materials for every kid
- Set time budgets, crawl hour, timezone

### Student

- No Google.
- Any parent creates the account: display name, username, password, grade.
- Student signs in with username + password and lands on **their** Today. No sibling switcher on a student session.
- Username unique in the app. Password hashed by Floot auth.
- Under-13: parents own the account. Student cannot invite adults, connect LMS, or delete the household.

### Session rules

- Parent session: household chrome (kids list, Materials, Status, Settings, Invites).
- Student session: Today, Session, their Materials. Log out only.
- Same device, two people: log out. No parent cookie under a student session.

## Invites

- `household_invite`: email, invited_by, expires_at (7 days), status pending/accepted/revoked/expired.
- Link is single-use. Wrong Google account (email mismatch) is rejected with a clear error.
- Pending invites list on Settings. Any parent can resend or revoke.
- Accepting Google SSO that already belongs to another household: blocked. One parent login, one household in v1.

## Email fan-out

Every household email goes to **all current guardians**, not just the creator.

Must fan out:

- Guardian invite (to the invitee only — obvious exception)
- Invite accepted / parent removed (all remaining)
- Schoology connected, needs reconnect, crawl missed, crawl summary (if enabled)
- Nightly status digest (when we turn email on)
- Student password reset confirmation
- “I'm stuck” optional ping

Use `@floot/email`. One template, N recipients from `app_user` where `role=parent` and household matches. No silent opt-out in v1 except leaving the household. Per-address mute is P1.

## Schoology access (separate from app login)

Three rungs. Same repo and crawl job either way.

### Rung A — district API (target)

Board-approved read-only materials API. Official tokens in Floot secrets.

### Rung B — parent portal, one household connection (v1 interim)

One `lms_connection` on the household (`scope=household`, `portal_role=parent`). Any guardian may start or finish the connect. Token in Floot secrets. Portal password is not stored. Map each Schoology child → student profile. 2pm job walks every mapped child.

If parent accounts cannot OAuth, Rung B waits on the board. No password-scrape mode.

### Rung C — upload only

Always on.

## Setup wizard (first parent)

1. Continue with Google.
2. Household name + timezone (default America/Chicago).
3. Add student: name, grade, username, password. Repeat.
4. Optional: invite another guardian (email).
5. Optional: Connect Schoology parent portal → map children.
6. Parent home.

Student first run: username + password → Today.

## School board blurb

Family study app. Read-only materials and due dates already visible to the parent portal. No submissions. No public-model training on district files. Official API beats a parent-token workaround.
