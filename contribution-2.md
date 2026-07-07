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

