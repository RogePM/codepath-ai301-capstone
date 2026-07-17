# Contribution 2: Copilot: Chart X-axis and tooltips show "Invalid Date" when using PostgreSQL

**Contribution Number:** 2  
**Student:** Rogelio Perez Montero  
**Issue:** https://github.com/backstage/community-plugins/issues/9540  
**Status:** Merged
**PR Link:** https://github.com/backstage/community-plugins/pull/9805  
---

## Why I Chose This Issue

I chose this issue because it is a good exercise in full-stack data normalization and TypeScript serialization. As a developer targeting backend and full-stack roles, understanding how different databases (like PostgreSQL versus SQLite) handle date objects before serializing them into JSON payloads is a critical real-world skill.

This also let me build on my existing knowledge of the Backstage Copilot plugin architecture. Tracking a data-parsing bug from the Knex database query layer all the way to the frontend React utility layer helped me strengthen my debugging skills and learn how to write safer, more resilient data-mapping functions.

---

## Understanding the Issue

### Problem Description
When the Backstage Copilot plugin is connected to a PostgreSQL database, the backend date columns are serialized into full ISO 8601 strings (e.g., `2026-05-26T00:00:00.000Z`). When this payload reaches the frontend, the `formatDay` utility blindly appends `T00:00:00` to the string before passing it to the JavaScript `Date` constructor, resulting in an "Invalid Date" error.

### Expected Behavior
The chart X-axis and tooltips should display correctly formatted date strings (e.g., "May 26") regardless of whether the backend database is PostgreSQL or SQLite.

### Current Behavior
The chart data renders, but the X-axis tick labels and tooltips display "Invalid Date" because the `Date` constructor fails to parse the malformed string.

### Affected Components
- **Backend:** The V2 metric getter methods in `plugins/copilot-backend/src/db/DatabaseHandlerV2.ts`, which returned raw Knex query results without normalizing the `day` column.
- **Frontend:** The `formatDay` utility function in `plugins/copilot/src/components/V2Dashboard/charts/chartUtils.ts`.

<img width="1970" height="1276" alt="image" src="https://github.com/user-attachments/assets/0e780464-a5bb-4e8c-a0c1-d3ee7d49d103" />

## Reproduction Process

### Environment Setup
The bug occurs specifically because of how the PostgreSQL (`pg`) driver handles database `DATE` columns compared to SQLite. Instead of spinning up a full live Postgres instance, I isolated and reproduced the exact failure using a targeted Jest unit test against the frontend utility.

### Steps to Reproduce
1. Opened a terminal in `workspaces/copilot`.
2. Ran a unit test passing a Postgres-style ISO string into the formatting utility: `expect(formatDay('2026-05-26T00:00:00.000Z')).toBe('May 26');`
3. Observed the crash: evaluating `new Date("2026-05-26T00:00:00.000ZT00:00:00")` produced `Invalid Date`.

### Reproduction Evidence
The automated test captured the regression failure (`Expected: "May 26", Received: "Invalid Date"`).

The core issue is a data-contract mismatch: when Knex runs on Postgres, the driver deserializes SQL `DATE` columns into JavaScript `Date` objects in memory. Express then serializes these to JSON as full ISO 8601 strings. The frontend `formatDay` utility expects a 10-character `"YYYY-MM-DD"` string and breaks when it receives the full timestamp instead.

---

## Solution Approach

### Analysis
- **Bug 1 (Backend API contract):** `DatabaseHandlerV2` already had a `normalizeDay()` private helper designed to handle JS `Date` objects and format them safely, but it was never called in the 7 primary V2 metric getter methods before the data was returned to the Express router.
- **Bug 2 (Frontend defensiveness):** `chartUtils.ts` assumed the backend would always send perfectly formatted data. By appending `T00:00:00` without validating the string length, it produced malformed timestamps like `"2026-05-26T00:00:00.000ZT00:00:00"`.
- **Bug 3 (Type safety in the fix itself):** My first pass at the backend fix mapped over rows and cast the result with `this.normalizeDay(row.day) as string`. An automated Copilot code review correctly flagged this: `normalizeDay()` returns `string | null`, so the cast silently discarded the possibility of `null` and could let a bad value leak into the API response instead of failing loudly.

### Proposed Solution (Defense in Depth)
- **Backend fix (root cause):** Mapped over the query results in `DatabaseHandlerV2.ts` (`getDailyTotals`, `getPrMetrics`, `getByFeature`, `getByIde`, `getByLanguageFeature`, `getByModelFeature`, `getByLanguageModel`, and `getIngestionLog`) and normalized the `day` column before returning the payload, guaranteeing the API outputs `"YYYY-MM-DD"` strings.
- **Type-safety fix (review feedback):** Replaced the unsafe `as string` cast with a new `normalizeRequiredDay()` helper. It calls `normalizeDay()` internally and throws a descriptive error if the result is `null`, rather than silently forwarding an invalid value. This makes an unexpected data issue visible immediately (as a clear error) instead of letting it surface later as an inexplicable frontend crash.
- **Frontend fix (safety net):** Updated `formatDay` in `chartUtils.ts` to slice the first 10 characters of the input string (`day.slice(0, 10)`) before parsing. This ensures the UI stays resilient even if an unnormalized ISO string ever reaches it from a future or third-party endpoint.

---

## Implementation Notes

### UMPIRE Implementation Plan
I applied the UMPIRE framework to keep the fix structured:

- **Understand:** The Postgres driver deserializes `DATE` columns into `Date` objects, which serialize to full ISO strings, breaking the frontend's string-manipulation utility.
- **Match:** Used the existing `normalizeDay` helper on the backend, added a stricter `normalizeRequiredDay` helper for non-nullable columns, and applied a defensive `.slice()` strategy on the frontend to handle varied date-string lengths.
- **Plan:**
  1. `DatabaseHandlerV2.ts` — map over query rows in the getter methods and normalize the `day` property.
  2. `chartUtils.ts` — update `formatDay` to extract `day.slice(0, 10)`.
  3. `chartUtils.test.ts` — add a regression test verifying that full ISO timestamps are safely formatted to `"May 26"`.
  4. `DatabaseHandlerV2.ts` — after automated review feedback, replace the `as string` cast with a `normalizeRequiredDay()` helper that throws on unexpected `null` values.
- **Implement:** Executed the plan across the database handlers and frontend utilities, keeping both layers of the stack consistent and safe.
- **Review:** Addressed the automated Copilot review comment about the unsafe type cast by introducing `normalizeRequiredDay()`, and separately cleaned up leftover debugging comments before merge.
- **Evaluate:** Verified the fix by running the full test suite (120 tests) and confirming no regressions were introduced.

### Progress Log

**Week 3:** Diagnosed the root cause by investigating how different database drivers handle the SQL `DATE` format. Wrote an isolated test script (`demo-invalid-date-bug-9540.mjs`) to reproduce the bug without needing a live Postgres container. Drafted the initial frontend `.slice()` safety net in `chartUtils.ts`.

**Week 4:** Realized that fixing only the frontend left the backend API contract leaking incorrect data types, and pivoted to a full "Defense in Depth" strategy by updating `DatabaseHandlerV2.ts` to normalize all raw Knex query results. Received automated review feedback flagging an unsafe type cast (`as string`) and resolved it by adding `normalizeRequiredDay()`. Also caught and unwound an accidental staging of a local debug script using a soft reset. Generated the Backstage changeset, opened the PR, and got it merged.

### Code Changes
- **Files modified:**
  - `plugins/copilot-backend/src/db/DatabaseHandlerV2.ts`
  - `plugins/copilot-backend/src/db/DatabaseHandlerV2.test.ts`
  - `plugins/copilot/src/components/V2Dashboard/charts/chartUtils.ts`
  - `plugins/copilot/src/components/V2Dashboard/charts/chartUtils.test.ts`
  - `.changeset/poor-plants-yell.md`
- **Key commits:**
- 
| Date   | Commit  | Summary                                              |
|--------|---------|-------------------------------------------------------|
|  07/14  |  eaebdbc3b | fix(copilot): normalize postgres date outputs and add defensive frontend parsing — core fix across DatabaseHandlerV2.ts, DatabaseHandlerV2.test.ts, chartUtils.test.ts, and changeset |
|  07/14   | 194c46b38 | Replace unsafe `as string` cast on normalizeDay() with a normalizeRequiredDay() helper   |

- **Approach decisions:** Submitted both frontend and backend fixes in a single PR because it eliminates the bug at the root while also adding resilience to the React UI against future endpoint changes. Chose to throw inside `normalizeRequiredDay()` rather than silently filtering out bad rows, since a `NOT NULL` column producing an invalid value indicates a real upstream data problem that should be surfaced, not hidden.

---

## Testing Strategy

### Unit Tests
- [x] **Frontend defensive parsing:** Added a regression test in `chartUtils.test.ts` verifying that `formatDay` correctly formats a full ISO 8601 timestamp (e.g., `2026-05-26T00:00:00.000Z` → `"May 26"`), in addition to the existing plain `"YYYY-MM-DD"` case.
- [x] **Backend API contract:** Updated `DatabaseHandlerV2.test.ts` to confirm all getter methods return normalized `"YYYY-MM-DD"` day strings, and that `normalizeRequiredDay()` throws when given an unparseable value.

### Manual Testing
Validated locally that `.slice(0, 10)` neutralizes the Postgres `pg` driver's ISO-timestamp behavior in the chart rendering.

---

## Pull Request

**PR Link:** https://github.com/backstage/community-plugins/pull/9805  
**Status:** Merged

---

## Learnings & Reflections

### Technical Skills Gained
- **Defense in depth:** Learned the value of fixing a bug at its root (the API layer) while also adding a defensive safety net on the frontend to guard against future regressions.
- **Type safety over convenience casts:** An automated review caught a spot where I'd used `as string` to satisfy the compiler instead of actually guaranteeing a non-null value at runtime. Replacing it with a helper that throws on invalid input (`normalizeRequiredDay`) taught me that a type cast is a promise to the compiler, not a runtime guarantee — and that promise needs to be backed by an actual check.
- **Helper function encapsulation:** Learned to refactor redundant data-processing logic by centralizing it into private helper functions. Instead of repeating normalization logic across seven getter methods, I reused the existing `normalizeDay` helper (and its stricter sibling) to keep the code consistent and easy to review.
- **PostgreSQL vs. SQLite drivers:** Deepened my understanding of how Knex interacts with different database engines — specifically how the `pg` driver deserializes `DATE` columns into JavaScript `Date` objects in memory, while SQLite returns strings directly.
- **Advanced Git workflow:** Practiced unstaging an accidental file addition using `git reset --soft HEAD~1` and `git restore --staged` to keep the commit history clean.

### Challenges Overcome
- Caught an accidental inclusion of a personal debugging script in a commit; learned to check `git status` carefully before committing.
- Addressed two rounds of review feedback — cosmetic (leftover debug comments) and substantive (the unsafe type cast) — without needing to open a new PR.

### What I'd Do Differently Next Time
I'll commit to running `git diff --staged` before every commit to catch stray files or debug comments before they get bundled into a commit, and I'll write the type-safe version of a helper function from the start rather than reaching for a cast and fixing it after review.

---

## Resources Used
- [Backstage Contributing Guidelines](https://github.com/backstage/community-plugins/blob/main/CONTRIBUTING.md)
- [Copilot Metrics API docs](https://docs.github.com/en/rest/copilot/copilot-metrics)
