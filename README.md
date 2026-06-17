# CodePath301

# Contribution [#8134]: [Bug]: Payee page associated rules count includes completed schedules

**Contribution Number:** #8134
**Student:** Anshu Anjna
**Issue:** https://github.com/actualbudget/actual/issues/8134
**Status:** Phase 2 - In Progress

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

I had to set up the Docker and install and login to it. 

I also had to look into installing yarn, which I have never used and 
when I tried running "npm install yarn", I got the error: "yarn: command not found." I asked claude to help me out and I had to run corepack enable after installing Node 22+, which makes the project's pinned Yarn available.

I also tried to activate my virtual environment: source venv/bin/activate. But that failed because I realised that Actual Budget is a TypeScript/Node monorepo and doesn't use python virtual environment. 

### Steps to Reproduce

1. Start Docker and open a budget file.
2. Create an account and a payee
3. Create a schedule for that payee, in the Schedules view, add a schedule, set its payee to "Tax Payment", and give it a near-term date with a limited or one-time recurrence so it can complete.
4. Post the scheduled transaction(s) so the schedule moves to the Completed state. Confirm it shows as completed in the Schedules view.
5. Confirm the payee has no manually-created rules: open the Rules view and verify there are none for "Tax Payment".
6. Open the Payees page (Manage Payees) and observe the rule count next to "Tax Payment".
7. Click into that payee's rules and compare the displayed count to the number of rules actually shown.
8. Expected: The rule count is 0, because there are no active rules (the only associated rule belongs to a completed schedule).
9. Actual: The rule count shows 1 (or one per completed schedule), even though clicking in reveals no rules.

### Reproduction Evidence

- **Commit showing reproduction:** https://github.com/anshuanjna/actual
- **Screenshots/logs:** [If applicable]
- **My findings:** The inflated count comes from the server, not the UI. The count is displayed in packages/desktop-client/src/components/payees/ManagePayees.tsx / PayeeTableRow.tsx via the ruleCount prop, but it is computed by a handler in packages/loot-core/src/server/. That handler counts all rules referencing a payee without excluding the hidden rules that back schedules. 

---

## Solution Approach

### Analysis

Schedules in Actual are implemented as hidden rules (each schedule has an associated rule carrying a link-schedule action that references the schedule's payee). The per-payee rule count computed on the server counts all rules referencing a payee and does not exclude rules belonging to schedules, in particular, rules belonging to completed schedules. As a result, a payee whose only associated rules are completed-schedule rules shows a non-zero count despite having no active, user-visible rules.

### Proposed Solution

Modify the server-side rule-counting logic so it excludes rules linked to completed schedules. Two candidate approaches, I will confirm the intended behavior with maintainers on the issue before coding:

(a) Exclude only rules linked to completed schedules (literal reading of the issue title).
(b) Exclude all schedule-linked rules from the count (if schedule rules were never meant to appear in this count at all).

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** The Payees page rule count is inflated because it counts the hidden rules that back completed schedules. Since schedules are implemented as rules in Actual, those rules are included in a payee's count even though the schedule is no longer active and the user sees no real rules. The count should reflect only active, user-relevant rules.

**Match:** link-schedule is the action type marking a rule as schedule-backed, find where Actual already distinguishes these rules elsewhere.

2. packages/loot-core/src/shared/schedules.ts already contains schedule-status logic (including "completed"); reuse its definition of completed.

3. Mirror the filtering style of the existing rule queries in loot-core/src/server/.

**Plan:** [Step-by-step implementation plan]
1. Locate the server handler in packages/loot-core/src/server/ that produces per-payee rule counts.
2. Modify the query/aggregation to exclude rules linked to completed schedules (approach (a) unless maintainers prefer (b)).
3. Adjust any affected types.
4. Add and update unit tests in loot-core covering the completed-schedule case.
5. Verify the frontend (ManagePayees/PayeeTableRow) needs no change once the server count is correct.

**Implement:** [Link to your branch/commits as you work]

**Review:** Self-review against CONTRIBUTING.md. Run yarn typecheck, yarn lint:fix, and yarn test from the repo root before pushing. Add a release note via yarn generate:release-notes (category: Bugfix). Match the project's commit-message and PR conventions, and include Fixes #8134 in the PR description.

**Evaluate:** Unit tests (below) plus manual verification using the reproduction steps, the previously inflated count now reads correctly.

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
