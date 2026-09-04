# Auth and household setup

Locked 2026-09-03.

Two different logins. Do not mix them up.

1. **This app** — how a parent or kid signs into AI Tutor.
2. **Schoology** — how the 2pm crawler reads school materials.

## App login

### Parent

- Sign in with **Google SSO** only (Floot `oauth-login`).
- First Google sign-in creates the household. That parent is owner.
- Additional parents: owner invites by email → they Google-SSO in and join the same household.
- No parent username/password in this app. Google is the parent door.

Parent can:

- Add / archive students
- Assign each student a **username + password** for this app
- Reset a student password
- Connect Schoology (parent portal) once for the household
- Map each Schoology child to a student profile
- See status for every kid
- Set time budgets, crawl hour, timezone

### Student

- No Google.
- Parent creates the account: display name, username, password, grade.
- Student signs in on the kitchen iPad with that username + password and lands on **their** Today. No kid switcher on a student session.
- Parent can still open the parent home and jump into a kid view on the parent session.
- Username is unique in the app. Password is hashed by Floot auth — we never store it plaintext.
- Under-13: parent owns the account. Student cannot change email, cannot connect LMS, cannot delete the household.

### Session rules

- Parent session: household chrome (kids list, Materials, Status, Settings).
- Student session: Today, Session, their Materials. No Settings beyond “log out.”
- Same device, two people: log out. We do not keep a parent cookie underneath a student session.

## Schoology access (separate from app login)

Three rungs. Climb as the district allows. Same repo and crawl job either way.

### Rung A — district API (target)

You talk to the board. They approve an app / parent-accessible materials API. We store the official tokens. Cleanest, most durable.

### Rung B — parent portal, one household connection (v1 interim)

You already have a **parent Schoology login that sees every child’s materials**. That is the connection we want, not each kid’s student login.

- One `lms_connection` on the **household**, role `parent_portal`.
- Parent completes Schoology OAuth (or district-issued key) once.
- Token lives in Floot secrets. Parent portal username may be stored as an identifier. **Portal password is not stored.**
- After connect, we list the children the parent can see. Parent maps “Schoology: Ava T.” → student profile Ava.
- 2pm job uses that one token, walks each mapped child, files into that child’s course folders.

If PowerSchool will not issue OAuth for parent accounts, Rung B is blocked until the board moves. We do not scrape the parent website with a saved password as the product.

### Rung C — upload only

Always on. Parent drops files into the kid’s folders. Ingest AI still runs. This is how the house works the week the token dies or the board says “not yet.”

## Setup wizard (parent, first run)

1. Continue with Google.
2. Household name + timezone (default America/Chicago).
3. Add student: name, grade, username, password.
4. Repeat for each kid.
5. Optional: Connect Schoology parent portal → map children.
6. Land on parent home.

Student first run: username + password → Today.

## What we will tell the school board (short)

We are a family study app. We want read-only access to course materials and due dates already visible to the parent portal. We will not submit work, will not scrape grades for a public product, will not train public models on district files. A district-approved API beats a parent-token workaround.
