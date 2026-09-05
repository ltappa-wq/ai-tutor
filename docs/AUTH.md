# Auth and household setup

Locked 2026-09-03. Co-parents locked 2026-09-03. Parent email/password added 2026-09-05.

Two different logins. Do not mix them up.

1. **This app** — how a parent or kid signs into AI Tutor.
2. **Schoology** — how the 2pm crawler reads school materials.

## App login

### Parents / guardians

Parents pick **either** door. Same household powers either way.

- **Google SSO** (Floot `oauth-login`)
- **Email + password** (Floot `auth`)

Rules:

- First successful parent login (either method) creates the household. That person is `creator` only for bookkeeping. **Access is equal.** There is no admin-only screen the second parent cannot open.
- Email is required for every parent. It is how invites and the 11pm digest find them.
- One email = one parent identity. They cannot join a second household with that email in v1.
- If they later sign in with Google using the same email they already registered, attach `google_sub` to that row. Do not create a second parent.
- Add a second or third guardian: any current parent enters their email → invite link → they accept with Google **or** email+password using that mailbox → same permissions as the first parent.
- Cap: 5 guardians per household.
- Parent passwords (when used) are hashed by Floot auth. Never stored on `app_user`.
- A guardian can leave the household. They cannot delete the household if they are the only remaining parent — transfer or add someone first. Any remaining parent can remove another parent (log it).

Forgot password: email reset to that guardian address. Google-only parents use Google’s account recovery, not ours.

Every parent can:

- Add / archive students
- Assign or reset a student username + password
- Invite / remove other guardians
- Connect or reconnect Schoology
- Map Schoology children to profiles
- See status and materials for every kid
- Set time budgets, crawl hour, timezone

### Student

- No Google. No email required.
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
- Link is single-use. The accepting account’s email must match the invite (Google email or the address they register).
- Pending invites list on Settings. Any parent can resend or revoke.
- An identity already in another household is blocked. One parent login, one household in v1.

## Email fan-out

Every household email goes to **all current guardians**, not just the creator.

Must fan out:

- Guardian invite (to the invitee only)
- Invite accepted / parent removed (all remaining)
- Schoology connected, needs reconnect, crawl missed, crawl summary (if enabled)
- Nightly status digest
- Student password reset confirmation
- Parent password reset (that address only)
- “I'm stuck” optional ping

Use `@floot/email`. Recipients: `app_user` where `role=parent`. No silent opt-out in v1 except leaving the household. Per-address mute is P1.

## Schoology access (separate from app login)

Unchanged. App login method does not change LMS rungs.

### Rung A — district API (target)

Board-approved read-only materials API. Official tokens in Floot secrets.

### Rung B — parent portal, one household connection (v1 interim)

One `lms_connection` on the household. Token in Floot secrets. Portal password is not stored.

### Rung C — upload only

Always on.

## Setup wizard (first parent)

1. Continue with Google **or** register with email + password.
2. Household name + timezone (default America/Chicago).
3. Add student: name, grade, username, password. Repeat.
4. Optional: invite another guardian (email).
5. Optional: Connect Schoology parent portal → map children.
6. Parent home.

Student first run: username + password → Today.

## School board blurb

Family study app. Read-only materials and due dates already visible to the parent portal. No submissions. No public-model training on district files. Official API beats a parent-token workaround.
