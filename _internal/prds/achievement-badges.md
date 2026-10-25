# PRD: Achievement Badges

Status: ready-for-agent

## Problem Statement

A student who finishes a course, passes every quiz in a section, or keeps up a daily learning habit gets no acknowledgment for it. There is nothing to look back on, and nothing that pulls a student back tomorrow. A returning student sees no sign that their past effort was noticed, and no visible next goal to aim for.

The primary goal of this feature is **retention**: badges should give students a reason to come back and keep going. Recognition (a durable record of achievement) is the secondary goal.

## Solution

Students earn permanent badges for learning milestones:

- **First Lesson** — completing their first lesson anywhere on the platform.
- **Course Complete** — one per course, for completing every lesson in that course.
- **Section Mastery** — one per module, for passing every quiz in that module.
- **Streak badges** — 3, 7 and 30 consecutive active days of learning, across all courses.

The moment a badge is earned, a toast celebrates it. A new private profile page shows the student's current and longest streak, every badge they have earned, and the badges they are working toward with visible progress ("Streak: 4/7 days", "2/4 quizzes passed"). The dashboard gets a card showing recent badges and the badge closest to unlocking, so a returning student immediately sees both what they have achieved and what to do next.

Students who completed work before launch receive their badges through a one-time backfill, dated to when they originally earned them.

## User Stories

### Earning badges

1. As a student, I want to earn a badge when I complete my first lesson, so that my first step on the platform feels acknowledged.
2. As a student, I want to earn a badge when I complete every lesson in a course, so that finishing a course feels like an achievement.
3. As a student, I want a separate Course Complete badge for each course I finish, so that my collection reflects everything I have finished.
4. As a student, I want to earn a Section Mastery badge when I have passed every quiz in a module, so that mastering a section's material is recognised.
5. As a student, I want Section Mastery to depend only on passing the quizzes, not on marking lessons complete, so that the rule is easy to understand and track.
6. As a student, I want modules without quizzes to not offer a Section Mastery badge, so that mastery badges always mean something.
7. As a student, I want to earn streak badges at 3, 7 and 30 consecutive active days, so that I have a reason to come back every day.
8. As a student, I want a day to count as active when I complete a lesson or submit a quiz attempt, so that real learning keeps my streak alive.
9. As a student, I want a failed quiz attempt to still count as activity, so that showing up and trying is rewarded.
10. As a student, I want just opening or scrubbing a video not to count as activity, so that streaks reflect deliberate learning.
11. As a student, I want activity in any of my courses to count toward a single streak, so that my habit is about learning, not one particular course.
12. As a student, I want "days" to follow my own local timezone, so that studying in the evening never counts as the wrong day.
13. As a student, I want my past active days to stay fixed even if I travel or my timezone changes, so that I never lose a streak retroactively.
14. As a student, I want my timezone detected automatically, so that I don't have to configure anything.
15. As a student, I want each streak badge to be earned only once, so that the next tier remains my goal.
16. As a student, I want earned badges to be permanent, so that breaking a streak or an instructor adding content never takes away something I earned.
17. As a student, I want to keep my Section Mastery badge even if the instructor later adds a new quiz to that module, so that my achievement stands as of when I earned it.
18. As a student, I want to keep my badges even if the instructor deletes the course, module or quiz they were for, so that my record survives course clean-ups.
19. As a student, I want a badge for deleted content to still show the original course or module name, so that I remember what I earned it for.
20. As a student, I want badges to be awarded the instant the qualifying action happens, so that the acknowledgment feels connected to the effort.
21. As a student, I want submitting the same action twice (for example a double click) to never award a badge twice, so that my collection stays accurate.
22. As a student, I want to earn badges in archived courses I am still enrolled in, so that archival doesn't stop my progress being recognised.

### Quiz passing

23. As a student, I want "passed" to mean meeting the quiz's configured passing score, so that the badge rules match what my instructor set.
24. As an instructor, I want the passing score I configure on a quiz to actually be enforced, so that my standards for mastery are respected.
25. As a student who previously saw "Passed!" on a quiz, I want that pass to still count toward Section Mastery, so that changing the grading rule doesn't take away my earlier result.

### Being notified

26. As a student, I want a toast when I earn a badge, so that I notice the achievement as it happens.
27. As a student, I want one toast per badge when I earn several at once, so that each achievement gets its moment.
28. As a student, I want at most three badge toasts in a row with an "and N more" link to my profile, so that I'm not flooded with notifications.

### Profile page

29. As a student, I want a private profile page showing my badges, so that I have something to look back on.
30. As a student, I want to see my current streak on my profile, so that I know where I stand today.
31. As a student, I want to see my longest-ever streak, so that a broken streak doesn't erase my best effort.
32. As a student, I want my current streak to stay alive until the end of today if I was active yesterday, so that I'm not shown a broken streak before I've had a chance to study today.
33. As a student, I want to see when I earned each badge, so that my collection doubles as a timeline of my learning.
34. As a student, I want to see the badges I haven't earned yet with my progress toward them, so that I know what to aim for next.
35. As a student, I want locked badges grouped by course, so that I can focus on one course at a time.
36. As a student, I want courses I haven't started to collapse to a single summary line, so that the page isn't overwhelmed by dozens of locked badges.
37. As a student, I want to see the next streak tier with my progress toward it, so that I'm motivated to come back tomorrow.
38. As a student, I want to reach my profile from the app's user menu, so that it's easy to find.
39. As a student, I want my profile to be visible only to me for now, so that I don't have to worry about privacy.

### Dashboard

40. As a returning student, I want a dashboard card showing my recently earned badges, so that I immediately see my effort was noticed.
41. As a returning student, I want the dashboard card to show the badge closest to unlocking, so that I know exactly what to do next.

### Existing students and data integrity

42. As a student who completed courses before launch, I want to receive the badges I already qualified for, so that I'm not penalised for being early.
43. As a student who completed work before launch, I want backfilled badges dated to when I originally qualified, so that my timeline is truthful.
44. As an active student at launch, I want my existing streak recognised from my past activity, so that I don't restart from zero on day one.
45. As an operator, I want the backfill to be safely re-runnable, so that a partial or repeated run never creates duplicate badges.
46. As an operator, I want backfilled streaks to only ever under-count, so that no student receives a badge they didn't earn.
47. As a student, I want finishing a course to also mark my enrollment as completed, so that the rest of the platform (such as the dashboard's completed state) reflects it.

### Eligibility

48. As an instructor previewing my own course, I want to not earn student badges, so that badges only reflect student learning.
49. As a platform owner, I want badges to require an enrollment in the course, so that badges can't be earned outside the normal learning flow.

## Implementation Decisions

### New achievements module (the single seam)

A new achievements service owns all badge and streak behaviour. It has three entry points:

- **Record a learning event** — takes the user, an event, and the current time. The event is either *lesson completed* (with the lesson) or *quiz attempted* (with the quiz and whether it passed). It:
  1. Appends the activity to the learning activity log, stamped with the user's local date at event time.
  2. Evaluates only the badges this event could affect: lesson completion checks First Lesson, Course Complete and streaks; quiz attempts check Section Mastery and streaks.
  3. Awards any newly qualified badges idempotently.
  4. When a course becomes fully complete, also marks the enrollment complete via the existing enrollment service.
  5. Returns the list of newly earned badges (empty when none).
- **Get achievement summary** — takes the user and the current time. Returns the current streak, longest streak, earned badges (with earned date and display title), locked badges with progress grouped by course (untouched courses collapsed to a summary), and the single badge closest to unlocking. Both the profile page and the dashboard card render from this.
- **Backfill achievements** — one-time operation taking the current time. Seeds the activity log from historic data and awards all qualifying badges with their earliest qualifying timestamps. Safe to re-run.

Internals (the badge registry, streak calculation, per-badge evaluators) are private to the module.

### Badge registry

Badge definitions are hard-coded in a typed registry in application code, not in the database. Each definition holds: badge type, display name, description, lucide icon, scope (global, per course, or per module), and how to compute progress toward it. Badge types:

- `first_lesson` (global)
- `course_complete` (per course)
- `section_mastery` (per module)
- `streak_3`, `streak_7`, `streak_30` (global)

### Badge rules

- **First Lesson:** the user has completed at least one lesson in a course they are enrolled in.
- **Course Complete:** every lesson in the course is completed by the user. Modules and lessons are evaluated as the course exists at evaluation time.
- **Section Mastery:** every quiz in the module has at least one attempt by the user with the stored `passed` flag set. A module with zero quizzes never awards this badge. Lesson completion is not required.
- **Streaks:** a streak is a run of consecutive distinct local dates in the activity log. The current streak is alive if the most recent active date is today or yesterday in the user's timezone; otherwise it is zero. Streaks are global across courses. Breaks are strict (no grace days, no freezes). Each streak badge is awarded once, the first time the threshold is reached.
- **Eligibility:** badges are awarded only for courses the user is enrolled in. Archived courses still award badges. Instructors previewing their own courses without an enrollment earn nothing.
- **Permanence:** once a row exists in the user badges table it is never removed by later events — not by a broken streak, added content, lesson progress resets, or content deletion.

### Schema changes

- **New `user_badges` table:** user, badge type, nullable course reference, nullable module reference, title snapshot (course or module title at time of earning, null for global badges), earned-at timestamp. Unique constraint across (user, badge type, course, module) makes awarding idempotent. Course and module references use set-null on delete so badges survive content deletion; the display falls back to the title snapshot marked as removed.
- **New `learning_activity` table:** user, activity type (`lesson_completed` | `quiz_attempted`), occurred-at (UTC ISO string, matching existing timestamp convention), and local date (YYYY-MM-DD in the user's timezone at the time of the event). The local date is stamped at write time and never recomputed. Index on (user, local date).
- **New nullable `timezone` column on `users`** holding an IANA timezone name. When null, UTC is used.

### Quiz scoring fix

Quiz scoring currently hardcodes `passed = score > 0.7` and ignores the quiz's configured passing score. Change it to `passed = score >= passingScore`, in both the scoring path that records attempts and the score lookup. Existing stored attempts are not re-graded; their `passed` flag stands.

### Route integration

- The lesson page action's `mark-complete` and `submit-quiz` intents call *record learning event* after the existing progress / scoring call, and include the newly earned badges in the action's return data.
- The lesson page shows one sonner toast per new badge, in sequence, capped at three plus an "and N more" toast linking to the profile — following the existing pattern where the quiz result toast is driven from fetcher data.
- **Timezone capture:** the authenticated app layout reads the browser's resolved IANA timezone and, when it differs from the stored value (or none is stored), posts it via a small fetcher to a resource route that updates the user's timezone. No settings UI.
- **New private profile route** for the signed-in user, rendering the achievement summary: streak stats, earned badges, then locked badges with progress grouped by course. Linked from the app's user menu. Not accessible for other users.
- **Dashboard card** rendering recent badges and the closest-to-unlock badge from the achievement summary.

### Backfill

A one-time script (alongside the existing seed script) calls *backfill achievements*:

- Seeds `learning_activity` from historic quiz attempts and lesson completion timestamps, using UTC for local dates (no historic timezone exists). Because lesson completion timestamps are overwritten on re-completion, this can only under-count activity.
- Awards First Lesson, Course Complete, Section Mastery and streak badges from the seeded and existing data, each dated to the earliest moment the user qualified.
- Wires up enrollment completion for courses that are fully complete.
- Is idempotent: a re-run creates no duplicate activity rows or badges.

## Testing Decisions

- **Good tests assert external behaviour only:** given a database state and a sequence of calls to the achievements module, what badges were returned and what the summary reports. Tests must not reach into the registry, evaluators or streak calculator directly, and must not assert on table contents that aren't observable through the module's entry points.
- **One seam:** all badge and streak behaviour is tested through the achievements module's three entry points — record learning event, get achievement summary, backfill achievements. The current time is passed in explicitly, so streak and timezone tests are deterministic without fake timers.
- **Scenarios to cover:**
  - Each badge type awarded at exactly its threshold and not before.
  - Section Mastery not offered for quiz-less modules; failed attempts don't count; a passing retake does.
  - Streak continuity and breaks across day boundaries in non-UTC timezones (e.g. activity at 23:30 local on consecutive days); current streak alive "yesterday", zero after a missed day; longest streak retained after a break.
  - Timezone change does not alter previously stamped days.
  - Repeated events and re-reaching a streak threshold do not duplicate badges.
  - Badges persist after a module/course/quiz is deleted, showing the snapshot title.
  - Badges persist after content is added to a completed course or mastered module.
  - No badges without enrollment; badges still awarded in archived courses.
  - Enrollment marked complete when Course Complete is earned.
  - Summary grouping: in-progress courses expanded, untouched courses collapsed, closest-to-unlock badge chosen correctly.
  - Backfill awards historic badges with original dates, seeds streaks, and is idempotent on re-run.
- **Quiz scoring:** the existing scoring module gets tests pinning `score >= passingScore` behaviour, including the boundary (score exactly equal to the passing score passes) and a non-default passing score.
- **Prior art:** existing colocated service tests (e.g. progress and enrollment service tests) — mock the database module with an in-memory SQLite created from the real migrations via the shared test setup helpers, seed base data, then call exported service functions.
- **Not unit-tested:** timezone capture, toasts, profile and dashboard UI — consistent with the repo's current practice of not testing route modules.

## Out of Scope

- Public or shareable student profiles (backlog #25 Student Profiles beyond the private badges page).
- Streak freezes, grace days or other streak forgiveness.
- Instructor- or admin-configurable badges.
- A timezone settings UI or manual override.
- Notifications beyond in-app toasts (email, in-app notification centre — backlog #23).
- XP, levels, leaderboards (backlog #19, #20).
- Badges that count repeats (e.g. "7-day streak ×3").
- A GitHub-style contribution calendar (the rest of backlog #7).
- Re-grading historic quiz attempts against configured passing scores.
- Fixing existing hard-delete orphaning of progress rows (only badges are made deletion-safe).

## Further Notes

- This feature delivers backlog #15 in full, plus the streak tracking slice of #7 and a private slice of #25; the backlog is annotated accordingly.
- Today, course completion is never actually recorded (the enrollment completion function is only called from tests); this feature wires it up as a side effect of Course Complete.
- Deleting a quiz currently cascades and deletes its attempts. Badges survive because they are stored independently with set-null references and a title snapshot, but progress toward *unearned* Section Mastery for that module recalculates against the remaining quizzes.
- The companion implementation plan lives at `_internal/plans/achievement-badges.md` for use with the `.sandcastle` `<prd> <plan>` workflow.
