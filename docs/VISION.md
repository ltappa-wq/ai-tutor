# Vision

## The problem

School-age kids (and the parents behind them) do not lack content. They lack a **daily operating system** for schoolwork:

- What is due, and in what order?
- How do I start when the assignment feels huge?
- How do I study for tomorrow without rereading the whole chapter?
- How does a parent know work happened without hovering for two hours?

Generic chatbots dump answers. That produces dependency, not skill. Existing homework tools are either content libraries, answer engines, or LMS clones. None of those is a tutor that runs the *day*.

## The product

**AI Tutor** is a daily academic coach for a student and a light command center for a parent.

Every school day the app should:

1. Assemble a **daily plan** from classes, assignments, tests, and time available.
2. Help the student **prepare** (warm-up, recap, quiz from notes/lectures).
3. **Guide homework** in small steps — Socratic, not "here's the answer."
4. Help **structure assignments** (outline, rubric check, draft critique) while keeping the student's own writing.
5. Close the loop with a short **end-of-day status** the parent can see.

## Who it is for

### Primary user — Student (v1: grades 6–12)

Needs: start help, sequencing, short sessions, low shame when stuck, ADHD-friendly pacing (timers, checklists, one next action).

### Secondary user — Parent / guardian

Needs: visibility without doing the homework. "What got done. What is still open. Where they got stuck."

### Non-user (v1)

Teachers, schools, district admins. Integrations with Canvas/Google Classroom can come later. v1 is family-first.

## Principles

1. **Coach, don't cheat.** The model scaffolds, questions, checks understanding, and refuses to hand over completed work that would be submitted as the student's own.
2. **One next action.** The screen always answers: what do I do *right now*?
3. **Short sessions beat hero sessions.** 15–25 minute blocks with a visible finish line.
4. **Parent visibility is a feature, not surveillance theater.** Status is factual and brief.
5. **Local reality first.** A family of a few students in a US household is the design center. Multi-tenant school SaaS is a later expansion, not the v1 shape.
6. **Spec-driven.** If it is not in the spec, it does not ship.

## What winning looks like (90 days)

- A student opens the app after school and can start work in under 60 seconds.
- Daily plan is trusted enough that they do not rebuild it in Notes or a paper planner.
- Homework sessions produce *progress artifacts* (checked steps, quiz scores, outline) not just a chat log.
- A parent can glance at tonight's status in under 30 seconds.
- We can add a second child without rewriting the product.

## Positioning (if this becomes a product)

Not "ChatGPT for homework."
Not another LMS.

**The daily tutor that runs the school night.**

Pricing later. Architecture now should not block a household subscription (parent account + N student profiles).

## Explicit non-vision (v1)

- We are not building a full curriculum.
- We are not replacing the teacher.
- We are not auto-submitting anything to a school portal.
- We are not a social network, gradebook, or college-counseling suite.
