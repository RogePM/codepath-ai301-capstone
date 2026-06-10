# Contribution [1]: Copilot: Migrate to the New GitHub Copilot Usage Metrics API #7360

**Contribution Number:** [1]  
**Student:** Rogelio Perez  
**Issue:** [[GitHub issue link]](https://github.com/backstage/community-plugins/issues/7360)  
**Status:** [Phase I] [ Complete]

---

## Why I Chose This Issue

I selected this specific issue within the Backstage community-plugins ecosystem because it presents a great opportunity to learn real-world API lifecycle management and master software deprecation workflows. My programming background is rooted heavily in JavaScript and TypeScript, and I want to use this experience to challenge myself with production-grade data refactoring and object manipulation tasks. Rather than building isolated standalone tools, this issue forces me to dig deep into an existing enterprise codebase, analyze how multiple files communicate, and learn how to safely transition live systems away from legacy third-party endpoints.

Working on this migration is a deliberate step toward doing better as a software engineer and expanding my asynchronous development capabilities. Navigating a massive, open-source framework like Backstage will teach me how to read and extend architecture designed by senior developers, write clean data validation schemas, and write resilient TypeScript code. Overcoming the technical challenges of parsing complex REST response models will not only solidify my problem-solving habits but also build the exact engineering maturity I need to succeed in my long-term academic and global career goals.
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

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

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
