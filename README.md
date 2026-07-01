# CodePath301

# Contribution [#3096]: [Bug]: [Test Failure] TC-00141: Files section appears to sort newest/oldest in the wrong order

**Contribution Number:** #3096
**Student:** Anshu Anjna
**Issue:** https://github.com/openedx/frontend-app-authoring/issues/3096
**Status:** Phase 2 - In Progress

---

## Why I Chose This Issue

I choose this issue because it matching my skills, like it uses TypeScript and JS, which I know through my coursework and projects. Also the bug is about a date sort returning results in the wrong order and it seems was well-scoped for a first contribution, with the maintainer already narrowing it to two likely root causes (a reversed comparator, or the "Newest"/"Oldest" labels mapped to the wrong sort direction).

Openedx is also a well-maintained, community-driven project with active Discord support and monthly releases, which means I'll have a supportive environment to work through my first open source contribution. The maintainer confirmed they would assign the issue to me once I could set up the dev environment and reproduce the bug.


---

## Understanding the Issue



### Problem Description

The Files section in Course Authoring sorts files incorrectly when using the date-based sorting options. The date sort order appears reversed, or the "Newest"/"Oldest" labels appear mapped to the wrong sort direction.

### Expected Behavior

Sorting by Newest should place the most recently uploaded file at the top of the list.
Sorting by Oldest should place the oldest uploaded file at the top of the list.

### Current Behavior

Sorting by Newest places the most recently uploaded file at the bottom of the list.
Sorting by Oldest places the most recently uploaded file at the top of the list.

### Affected Components

The Files & Uploads page within the course authoring MFE (the component(s) responsible for the file list and its date-sort dropdown).

---

## Reproduction Process

### Environment Setup

1. Forked and cloned openedx/frontend-app-authoring.
2. Began local frontend setup: nvm use to match the Node version in .nvmrc, then npm install.
3. Hit a fedx-scripts: command not found error on npm start, traced to an incomplete dependency install; resolved by doing a clean reinstall (rm -rf node_modules package-lock.json && npm install) on the correct Node version.
4. Tutor (the recommended Open edX backend dev environment) setup is in progress to allow full UI reproduction.

### Steps to Reproduce

1. Go to Studio / Course Authoring.
2. Open a course.
3. Go to the Files section.
4. Upload a new file and confirm it appears in the list.
5. Change the sort option to Newest and check the new file's position.
6. Change the sort option to Oldest and check the position again.

### Reproduction Evidence

- **Commit showing reproduction:** 
- **Screenshots/logs:** [If applicable]
- **My findings:** 

---

## Solution Approach

### Analysis



### Proposed Solution



### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** 

**Match:** 


**Plan:** [Step-by-step implementation plan]
1. 

**Implement:** [Link to your branch/commits as you work]

**Review:** 
**Evaluate:** 

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

### Week [3] Progress

My chosen issue is now closed due to someone else resolving the bug and now I'm trying to find a new issue and I'm hoping to find an issue that relates to my topics of interest: Biotech, AI in healh, and ML projects. Overall, couldn't do much this weekend as I'm trying to find an open issue, which is hard since a lot of them are taken or are issues with red flags. 

### Week [4] Progress

I have found an issue and now I'm reproducing the issue and analyizing the possible solutions. 

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
