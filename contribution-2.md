# Contribution 2: Copilot: Chart X-axis and tooltips show "Invalid Date" when using PostgreSQL

**Contribution Number:** 2  
**Student:** Rogelio Perez Montero  
**Issue:** https://github.com/backstage/community-plugins/issues/9540
**Status:** Phase I Complete

---

## Why I Chose This Issue

I chose this issue because it is a perfect exercise in full-stack data normalization and TypeScript serialization. As a developer targeting backend and full-stack roles, understanding how different databases (like PostgreSQL versus SQLite) handle date objects before serializing them into JSON payloads is a critical real-world skill. 

Furthermore, this allows me to build on my previous knowledge of the Backstage Copilot plugin architecture. By tracking a data parsing bug from the Knex database query level all the way to the frontend React utility layer, I can strengthen my debugging skills and learn how to write safer, more resilient data mapping functions.

---

## Understanding the Issue

### Problem Description
When the Backstage Copilot plugin is connected to a PostgreSQL database, the backend date columns are serialized into full ISO 8601 strings (e.g., `2026-05-26T00:00:00.000Z`). When this payload reaches the frontend, the `formatDay` utility blindly appends `T00:00:00` to the string before passing it to the JavaScript `Date` constructor, resulting in an "Invalid Date" error.

### Expected Behavior
The chart X-axis and tooltips should display correctly formatted date strings (e.g., "May 26") regardless of whether the backend database is PostgreSQL or SQLite.

### Current Behavior
The chart data renders, but the X-axis tick labels and tooltips display "Invalid Date" because the `Date` constructor fails to parse the malformed string.

### Affected Components
- **Backend:** The route handlers in `service/router.cjs.js` returning raw Knex query results without normalizing the date.
- **Frontend:** The `formatDay` utility function in `chartUtils.esm.js`.

<img width="1970" height="1276" alt="image" src="https://github.com/user-attachments/assets/0e780464-a5bb-4e8c-a0c1-d3ee7d49d103" />

## Reproduction Process

### Environment Setup
The bug specifically occurs due to how the PostgreSQL (`pg`) driver handles database `DATE` columns compared to SQLite. Instead of needing to spin up a full live Postgres instance, I isolated and reproduced the exact failure logic using a targeted Jest unit test against the frontend utility.

### Steps to Reproduce
1. Open the terminal in `workspaces/copilot`.
2. Run a unit test passing a Postgres-style ISO string into the formatting utility: `expect(formatDay('2026-05-26T00:00:00.000Z')).toBe('May 26');`
3. Observe the crash: evaluating `new Date("2026-05-26T00:00:00.000ZT00:00:00")` results in `Invalid Date`.

### Reproduction Evidence
* **Automated Test Output:** Successfully captured the regression failure (`Expected: "May 26", Received: "Invalid Date"`).
* **My findings:** The core issue is a data contract mismatch. When Knex runs on Postgres, the driver deserializes SQL `DATE` columns into JavaScript `Date` objects in memory. Express serializes these to JSON as full ISO 8601 strings. The frontend `formatDay` utility expects a 10-character `"YYYY-MM-DD"` string and breaks when it receives the full timestamp.

---

## Solution Approach

### Analysis
* **Bug 1 (Backend API Contract):** `DatabaseHandlerV2` has an existing `normalizeDay()` private helper designed specifically to handle JS `Date` objects and format them safely. However, it is never called in the 7 primary V2 metric getter methods before passing the data to the Express router.
* **Bug 2 (Frontend Defensiveness):** `chartUtils.ts` assumes the backend will always send perfectly formatted data. By appending `T00:00:00` without validating the string length, it creates malformed timestamps like `"2026-05-26T00:00:00.000ZT00:00:00"`.

### Proposed Solution (Defense in Depth)
* **Backend Fix (Root Cause):** Map over the query results in `DatabaseHandlerV2.ts` (e.g., `getDailyTotals`, `getPrMetrics`, etc.) and invoke `this.normalizeDay(row.day)` before returning the payload. This guarantees the API strictly outputs `"YYYY-MM-DD"`.
* **Frontend Fix (Safety Net):** Update `formatDay` in `chartUtils.ts` to implement defensive parsing by slicing the string (`day.slice(0, 10)`). This ensures the UI will never crash again, even if an unnormalized ISO string sneaks through a future API endpoint.

---

## Implementation Plan

* **Understand:** The Postgres driver deserializes dates to objects, which serialize to ISO strings, breaking the frontend string-manipulation utility.
* **Match:** Use the existing `normalizeDay` helper on the backend, and apply a standard string truncation strategy (`.slice`) on the frontend to gracefully handle varied date string lengths.
* **Plan:**
  * `DatabaseHandlerV2.ts` — Map over query rows in the getter methods and normalize the `day` property.
  * `chartUtils.ts` — Change `formatDay` to extract `day.slice(0, 10)`.
  * `chartUtils.test.ts` — Write a regression test verifying that full ISO timestamps are safely formatted to `"May 26"`.
