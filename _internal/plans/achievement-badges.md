# Plan: Achievement Badges

PRD: `_internal/prds/achievement-badges.md`

Each phase is one task and a tracer-bullet vertical slice: schema, service, UI and tests together, demoable on its own. Work phases top to bottom; pick the first phase whose status is `ready-for-agent` and whose blockers are all `done`.

When you finish a phase, in the same commit: tick its acceptance criteria and change its **Status** to `done`.

All badge and streak behaviour is tested through the achievements module's entry points (record learning event, get achievement summary, backfill achievements), passing the current time in explicitly. Follow the existing colocated service tests that mock the database module with the in-memory test database.

---

## Phase 1 — Quiz passing uses the configured passing score

**What to build:** A quiz attempt passes when its score meets the quiz's configured passing score (`score >= passingScore`) instead of the hardcoded `score > 0.7`. The lesson page's pass/fail result reflects the configured score. Historic attempts keep their stored `passed` flag — no re-grading.

**Blocked by:** None — can start immediately.

**Status:** ready-for-agent

- [ ] Both the attempt-recording path and the score lookup in quiz scoring use the quiz's passing score
- [ ] New tests for quiz scoring cover: a score exactly equal to the passing score passes; just below fails; a non-default passing score (e.g. 0.5 and 0.9) is respected
- [ ] Existing stored attempts are not modified
- [ ] `npm run test` and `npm run typecheck` pass

---

## Phase 2 — Tracer bullet: First Lesson badge end to end

**What to build:** A student who completes their first lesson in an enrolled course earns the First Lesson badge, sees a toast for it on the lesson page, and can see it on a new private profile page reached from the user menu. This phase lays the foundation every later badge builds on:

- `user_badges` table (user, badge type, nullable course ref, nullable module ref, title snapshot, earned-at; unique on user + badge type + course + module).
- `learning_activity` table (user, type `lesson_completed` | `quiz_attempted`, occurred-at UTC, local date; indexed on user + local date). Local date is UTC for now — timezone arrives in Phase 5.
- Hard-coded typed badge registry (type, name, description, lucide icon, scope), containing `first_lesson` for now.
- New achievements service with **record learning event** (logs activity, evaluates badges affected by that event, awards idempotently, returns newly earned badges) and **get achievement summary** (earned badges for now).
- The lesson page action's `mark-complete` and `submit-quiz` intents call record learning event after the existing call and return new badges in their action data.
- Toasts: one per new badge in sequence, capped at three plus an "and N more" toast linking to the profile.
- Private profile route listing earned badges with earned dates; linked from the app's user menu; only shows the signed-in user.

**Blocked by:** None — can start immediately.

**Status:** ready-for-agent

- [ ] Completing a first lesson in an enrolled course returns the First Lesson badge from record learning event; later completions return nothing new
- [ ] Repeating the same event (double submit) never creates a duplicate badge
- [ ] A user with no enrollment in the course earns no badge
- [ ] Both lesson completions and quiz attempts (pass or fail) are written to the activity log
- [ ] The lesson page shows a toast when a badge is earned, with the three-plus-"and N more" cap
- [ ] The profile page lists earned badges and is linked from the user menu
- [ ] Migration generated; `npm run test` and `npm run typecheck` pass

---

## Phase 3 — Course Complete badge

**What to build:** Completing every lesson in an enrolled course awards a per-course Course Complete badge (toast + profile) and marks the enrollment complete. Badges store a title snapshot and use set-null references, so they survive the course being deleted and display the snapshot title marked as removed.

**Blocked by:** Phase 2

**Status:** ready-for-agent

- [ ] The badge is awarded on the lesson completion that completes the course, and not before
- [ ] One badge per course; finishing two courses yields two badges
- [ ] The enrollment is marked complete at the same moment
- [ ] Adding a lesson to the course afterwards does not remove the badge
- [ ] Archived courses still award the badge to enrolled students
- [ ] After the course is deleted, the badge still appears in the summary with its snapshot title, marked as removed
- [ ] `npm run test` and `npm run typecheck` pass

---

## Phase 4 — Section Mastery badge

**What to build:** Passing every quiz in a module (by stored `passed` flag) awards a per-module Section Mastery badge (toast + profile). Lesson completion is not required. Modules with no quizzes never award it. The badge survives deletion of its module or quizzes.

**Blocked by:** Phase 1, Phase 2

**Status:** ready-for-agent

- [ ] The badge is awarded on the quiz attempt that passes the module's last unpassed quiz, and not before
- [ ] Failed attempts don't count; a later passing retake does
- [ ] A module with zero quizzes never awards the badge
- [ ] Adding a quiz to a mastered module does not remove the badge
- [ ] After the module is deleted, the badge still appears with its snapshot title, marked as removed
- [ ] `npm run test` and `npm run typecheck` pass

---

## Phase 5 — Streaks

**What to build:** Students build a global daily learning streak across all courses and earn 3-, 7- and 30-day streak badges, each once. Days follow the student's own timezone, captured silently from the browser. The profile shows current and longest streak.

- Nullable `timezone` column on users (IANA name); UTC when unknown.
- The authenticated app layout posts the browser's resolved timezone to a small resource route when it differs from the stored value.
- Activity local dates are stamped in the user's timezone at write time and never recomputed.
- Current streak is computed live from distinct local dates: alive if the latest active day is today or yesterday in the user's timezone, otherwise zero. Longest streak computed the same way.
- Strict breaks — no grace days.

**Blocked by:** Phase 2

**Status:** ready-for-agent

- [ ] Streak badges are awarded at exactly 3, 7 and 30 consecutive active days, and each only once (re-reaching a threshold after a break awards nothing new)
- [ ] Activity at 23:30 local on consecutive days in a non-UTC timezone counts as consecutive days
- [ ] The current streak is still alive the day after the last activity, and zero after a missed day
- [ ] Longest streak is retained after a break
- [ ] Changing a user's timezone does not alter previously stamped days
- [ ] The profile shows current and longest streak; timezone is captured without any UI
- [ ] Migration generated; `npm run test` and `npm run typecheck` pass

---

## Phase 6 — Locked-badge progress and dashboard card

**What to build:** The profile shows the badges a student is working toward, with progress ("2/4 quizzes passed", "Streak: 4/7 days", "5/12 lessons"), grouped by course: earned first, then in-progress, with courses the student hasn't started collapsed to a single summary line. The dashboard gets a card with recently earned badges and the single badge closest to unlocking.

**Blocked by:** Phase 3, Phase 4, Phase 5

**Status:** ready-for-agent

- [ ] The achievement summary returns locked badges with progress, grouped by enrolled course
- [ ] Courses with no progress are collapsed to a summary (e.g. "0/8 sections mastered")
- [ ] The summary identifies the badge closest to unlocking, including the next streak tier
- [ ] The profile renders the grouped progress; the dashboard card renders recent badges and the closest-to-unlock badge
- [ ] `npm run test` and `npm run typecheck` pass

---

## Phase 7 — One-time backfill

**What to build:** A one-time script, alongside the existing seed script, that gives students who learned before launch the badges they already earned. It seeds the activity log from historic quiz attempts and lesson completion timestamps (local dates in UTC), awards all qualifying First Lesson, Course Complete, Section Mastery and streak badges dated to the earliest moment each was qualified for, and marks fully completed enrollments complete.

**Blocked by:** Phase 3, Phase 4, Phase 5

**Status:** ready-for-agent

- [ ] Historic course, section and first-lesson badges are awarded with their original qualifying dates
- [ ] Historic streak badges are awarded from seeded activity; seeding can only under-count
- [ ] Section Mastery uses the stored `passed` flag on historic attempts
- [ ] Running the backfill twice creates no duplicate activity rows or badges
- [ ] An npm script runs it; `npm run test` and `npm run typecheck` pass
