# Contribution [1]: Copilot: Migrate to the New GitHub Copilot Usage Metrics API #7360

**Contribution Number:** [1]  
**Student:** Rogelio Perez  
**Issue:** [[copilot-metrics- Legacy API]](https://github.com/backstage/community-plugins/issues/7360)  
**Status:** [Phase II] [ Complete]

---

## Why I Chose This Issue

I selected this specific issue within the Backstage community-plugins ecosystem because it presents a great opportunity to learn real-world API lifecycle management and master software deprecation workflows. My programming background is rooted heavily in JavaScript and TypeScript, and I want to use this experience to challenge myself with production-grade data refactoring and object manipulation tasks. Rather than building isolated standalone tools, this issue forces me to dig deep into an existing enterprise codebase, analyze how multiple files communicate, and learn how to safely transition live systems away from legacy third-party endpoints.


---

## Understanding the Issue

### Problem Description

The GitHub Copilot integration plugin within the Backstage ecosystem currently relies on a legacy metrics endpoint to fetch telemetry data. GitHub officially sunset this old REST endpoint on April 2, 2026, the plugin is no longer able to capture or display Copilot data. The entire data-fetching mechanism must be refactored to consume the newly introduced GitHub Copilot Usage Metrics REST API.

### Expected Behavior

The plugin's TypeScript backend service should successfully construct secure requests to the updated GitHub API endpoint, handle authentication tokens cleanly, and dynamically parse the new JSON report payload properties into the application's internal reporting layer.

### Current Behavior

The integration modules point to a completely deprecated and unresponsive API path structure, preventing the plugin from pulling telemetry logs and causing the frontend metrics dashboard to break.

### Affected Components

- /plugins/copilot-backend (The core TypeScript backend service handling external network calls and data streams)
- TypeScript interface contracts and data models mapping the incoming JSON payload fields

---

## Reproduction Process

### Environment Setup
- **Development Path:** Initiated local development via terminal because Dev Containers glitched and didn't let me open  the app. I executed the workspace using Backstage's vendored Yarn engine directly via Node (`node ..\..\.yarn\releases\yarn-4.14.1.cjs start`).
- **Challenges Resolved:**
- 1. **Config Crash:** The frontend initially crashed with `Missing required config value at 'app.title'`. I diagnosed that the local `app-config.yaml` was wiped. I restored the core `app`, `backend`, and `catalog` blocks.
  2. **Auth Wall Bypass:** The Copilot dashboard is gated behind GitHub Enterprise credentials. I bypassed this by passing a dummy token (`ghp_fake_token_for_local_dev`) and configuring `defaultView: organization`.
  3. **API Rate/Crash Loop:** To prevent the background worker from immediately crashing against the dead GitHub API on boot, I added `initialDelay: seconds: 600` to the schedule config.
  4. **Database Seeding:** Since the frontend reads from a local SQLite DB, I created and ran a custom `seed-fake-data.js` script to inject 14 days of mock data into `copilot_daily_totals` for `entity_id=my-fake-org`. 

**Active Working Branch:** https://github.com/RogePM/community-plugins/tree/fix-copilot-metrics-api-7360

### Steps to Reproduce
1. Read the internal documentation and codebase paths, noting that `src/service/router.ts` splits routes between `/legacy/*` and `/v2/*`.
2. Configure `app-config.yaml` with a dummy GitHub token and a 10-minute scheduler delay.
3. Start the local server using `node ..\..\.yarn\releases\yarn-4.14.1.cjs start` inside of community-plugins\workspaces\copilot to generate the SQLite database schema.
4. Stop the server and seed the local `database/backstage_plugin_copilot.db` with fake metrics by running `node seed-fake-data.js`.
5. Restart the server and navigate to `http://localhost:3000/copilot`.
6. **Expected:** Based on the issue ticket, I expected the background ingestion scraper to eventually fail and throw `404/410 Gone` errors attempting to hit the deprecated `/copilot/metrics` API (which sunset on April 2, 2026).
7. **Actual :** The app successfully mounted and didn't crash. To investigate why the legacy code wasn't failing, I dug into the Git history using `git log --oneline --all -- workspaces/copilot/`. I discovered the issue was **already fixed upstream.** Commit `95bf1ed02` (PR #9405), merged on June 11, 2026 by Scott Guymer, had just executed this exact API migration.

---
### Reproduction Evidence

- **Commit showing reproduction:** 👉 https://github.com/RogePM/community-plugins/tree/fix-copilot-metrics-api-7360
- **Screenshots/logs:** N/A (Terminal output verified via Git Log)
- **My findings:** The codebase search verified that `GithubClient.ts` was deleted and replaced by `GithubClientV2.ts`. The Git log confirmed the migration was merged just a few days prior to my environment setup.

---

## Solution Approach

### Analysis

The root cause of the original issue was that GitHub officially deprecated the legacy `/copilot/metrics` REST API on April 2, 2026. The Backstage ingestion engine was hardcoded to hit this dead endpoint. Without a migration, background scraper tasks fail with network exceptions, leaving enterprise dashboards completely blank.

### Proposed Solution (How it was solved upstream)

The solution required rewriting the backend client to use the new GitHub Usage Metrics API (`/copilot/metrics/reports/enterprise-1-day`), updating the `X-GitHub-Api-Version` header to `2026-03-10`, and adjusting the TypeScript interfaces to process the new signed download URLs correctly.

### Implementation Plan (UMPIRE framework)

*Because this issue was resolved upstream during my investigation phase, this UMPIRE plan serves as an architectural post-mortem of the merged fix, proving my understanding of the required solution.*

* **Understand:** The objective was to replace the deprecated `/copilot/metrics` endpoint with the modern `/copilot/metrics/reports/*` API to maintain dashboard functionality and prevent scraping crashes.

* **Match:** By reviewing the diff of commit `95bf1ed02`, I confirmed the maintainers used the exact strategy I anticipated by utilizing `GithubClientV2.ts` to construct the new request headers.

* **Plan & Implement:** 1. **Deleted Legacy Client:** The old `GithubClient.ts` (which made `GET /enterprises/{slug}/copilot/metrics` calls) was entirely deleted from `src/client/`.
    2. **Promoted V2 Client:** `GithubClientV2.ts` became the primary driver, successfully routing to the modern `GET /enterprises/{slug}/copilot/metrics/reports/enterprise-1-day`.
    3. **Updated Headers:** The Octokit API version header was successfully bumped from `2022-11-28` to `2026-03-10`.
    
* **Review:** The upstream PR adhered to strict TypeScript typing and included a `changeset` file to log the version bump.

* **Evaluate:** The test suite (`GithubClientV2.test.ts`) was updated with new Jest mocks to simulate the payload of the 2026-03-10 API version, ensuring no regressions.
### Analysis

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
