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
2. Matched the required Node version (Node 24) using nvm use with the repo's .nvmrc.
3. Hit fedx-scripts: command not found on first run. Diagnosed as an incomplete dependency install; resolved with a clean reinstall.
4. Hit an ERESOLVE peer-dependency conflict (oxlint vs oxlint-tsgolint). Resolved by restoring the repo's original package-lock.json and using npm ci (installs the exact locked dependency tree) instead of npm install.
5. Installed Tutor (the official Docker-based Open edX dev environment) in a Python virtual environment and ran tutor dev launch.
6. First launch failed at the MySQL initialization job — the database container was still initializing when the setup job timed out after 10 connection attempts.
7. Later, the LMS returned connection resets. Diagnosed by reading the Django traceback in tutor dev logs lms: Table 'openedx.waffle_switch' doesn't exist — meaning migrations had never run because the interrupted launch never completed initialization.
8. Re-ran tutor dev launch, allowing the full migration phase to complete. Platform initialized successfully with all services running (LMS, CMS/Studio, MFE server, MySQL, MongoDB, Redis, Meilisearch).
10. Created a local superuser (tutor dev do createuser --staff --superuser) and imported the demo course (tutor dev do importdemocourse).

### Steps to Reproduce

1. Go to Studio / Course Authoring.
2. Open a course.
3. Go to the Files section.
4. Upload a new file and confirm it appears in the list.
5. Change the sort option to Newest and check the new file's position.
6. Change the sort option to Oldest and check the position again.

### Reproduction Evidence

- **Commit showing reproduction:** I haven't yet sent a message to the Maintainer as I'm rechecking the issue. 
- **Screenshots/logs:** This image shows my new file upload and when the filter is set to Newest, my file is on the top and later when the filer is set to Oldest, my file is on the bottom. <img width="1452" height="681" alt="Screenshot 2026-07-07 at 10 18 33 PM" src="https://github.com/user-attachments/assets/6667eb3c-10a5-4ae3-8d8c-1c5396c6867f" />

- **My findings:** On my devstack, the bug did not reproduce. After uploading a new file: sorting by Newest correctly placed my file first; sorting by Oldest correctly placed it last. This is the expected (correct) behavior, not the buggy behavior described in the issue. My test ran against the MFE version packaged in Tutor's pre-built image (openedx-mfe:21.0.0-indigo), because my local repository was not yet bind-mounted at test time. The issue was filed against current development code during Verawood release testing, so this result does not yet confirm the bug is fixed on master.

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

Committed to #3096. Commented on the issue to claim it. the maintainer confirmed they would assign it to me once I reproduce it on a devstack. 

### Week [5] Progress
Completed the full Tutor devstack setup, including diagnosing and fixing two real failures: a MySQL initialization timeout on first launch, and a missing-migrations state (diagnosed from a Django ProgrammingError: Table 'openedx.waffle_switch' doesn't exist traceback in the LMS logs) fixed by re-running the launch to completion. Created an admin user, imported the demo course, and attempted reproduction. Result: the bug did not reproduce — sort behavior was correct on the version I tested. Identified that I tested Tutor's packaged release image rather than current master. I have to now bind-mount my repo and retesting on master before reporting findings to the maintainer.


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
