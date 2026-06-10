# CodePath301

# Contribution [#8134]: [Bug]: Payee page associated rules count includes completed schedules

**Contribution Number:** #8134
**Student:** Anshu Anjna
**Issue:** https://github.com/actualbudget/actual/issues/8134
**Status:** Phase I - In Progress

---

## Why I Chose This Issue

I choose this issue because it matching my skills, like it uses TypeScript, which I know through my coursework and projects. I was also drawn to the fact that the bug involves real data logic, specifically how the app queries and filters rules associated with payees, which connects to the data science concepts I've been studying. Actual Budget is also a well-maintained, community-driven project with active Discord support and monthly releases, which means I'll have a supportive environment to work through my first open source contribution.

This issue is a good fit for my learning goals because it will push me to understand how a real-world TypeScript/React codebase structures its data layer, specifically how rules and schedules relate to each other in the database. I'm hoping to walk away with a clearer understanding of how to trace a UI bug back to its source in a query or filter, and how to write a clean fix with proper testing.

---

## Understanding the Issue

### Problem Description

On the Payees page in Actual Budget, each payee shows a count of how many rules are associated with it. This count is wrong, it includes completed schedules, which are schedules that have already finished running. Completed schedules are no longer active, so they shouldn't be counted as associated rules. The bug was discovered when a user noticed some payees showed a rules count but had no actual rules attached when they clicked in to view them.

### Expected Behavior

The rules count shown next to each payee on the Payees page should only reflect active, non-completed rules. Completed schedules should be excluded from this count entirely.

### Current Behavior

The rules count incorrectly includes completed schedules. A user noticed some counts had no rules attached when viewing them, and after deleting a rule, the associated count dropped by more than expected, indicating completed schedules were being counted. This means the number shown is inflated and misleading.

### Affected Components

The bug lives in the data layer that computes the rules count for payees. Likely affected areas:

The query or filter that fetches rules associated with a payee (probably in packages/loot-core/src/server/ — where Actual's backend data logic lives)
Possibly the PayeeTable or ManagePayees component on the frontend that displays the count

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
