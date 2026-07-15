# CodePath301

# Contribution [#3096]: [Bug]: [Test Failure] TC-00141: Files section appears to sort newest/oldest in the wrong order

**Contribution Number:** #3096
**Student:** Anshu Anjna
**Issue:** https://github.com/openedx/frontend-app-authoring/issues/3096
**Status:** Phase 3 - In Progress

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

- **Commit showing reproduction:** https://github.com/openedx/frontend-app-authoring/issues/3096 

Hi @bradenmacdonald, I set up devstack and reproduced the issue. I also realized that the issue is more prevalent when the files are uploaded one after the other quickly(< 1 minute). I noticed that when I uploaded an file and waited for 5-10 minutes and then uploaded another file, the filtering was accurate and there were no issues. 

For example, these files are under the Newest filter and this is following the correct guidelines. (I uploaded file1, file2, file3, respectively in that order).

<img width="1460" height="634" alt="Image" src="https://github.com/user-attachments/assets/e58869d5-8b09-42bd-92cf-ec6eeb371e40" />

But now, when I switch the filter settings to Oldest, the files got scrambled, but they are all located at the bottom of the list. 
<img width="1443" height="311" alt="Image" src="https://github.com/user-attachments/assets/b9042684-9039-4fda-92bd-af6c866c157f" />


And when I went back to the filter setting of Newest, the files came to the top but were also scrambled. 
<img width="1465" height="648" alt="Image" src="https://github.com/user-attachments/assets/d34f49e9-547f-4d21-a84a-7151dd5070b2" />

Another issue is that when the filering is set to Oldest and then a new file is uploaded, the file is at the top of the list, even though it should be on the bottom. But after changing the filtering setting, the file is in the correct position. Example: 

<img width="1463" height="739" alt="Image" src="https://github.com/user-attachments/assets/9ebe3950-35c3-478b-b9a8-d7e535328d58" />

Overall, I understand the issue reproduction and will be working on the solution to this problem. I'll take a look at possible solutions and also write additional unit tests. Thank you! Let me know if there is anything else I should look into for this issue! 

- **Screenshots/logs:** This image shows my new file upload and when the filter is set to Newest, my file is on the top and later when the filer is set to Oldest, my file is on the bottom. <img width="1452" height="681" alt="Screenshot 2026-07-07 at 10 18 33 PM" src="https://github.com/user-attachments/assets/6667eb3c-10a5-4ae3-8d8c-1c5396c6867f" />

- **My findings:** The bug isn't exactly as defined by the maintainers because the bug is more present if the files are uploaded quickly and without much wait time between uploads(< 1 min). Also since the site doesn't reload after each upload, after changing the filter settings from Newest to Oldest and vice versa, the order of the files are messed up in terms of which file you uploaded first for the most recent uploads. the bug doesn't seem to affect the older uploaded files. The main problem is that the when the files are uploaded at the same time, then they can interchange the order they are in. 

---

## Solution Approach

### Analysis
The actual bug is: the upload timestamp is only stored and compared at hour:minute, not down to the second. When two or more files are uploaded within the same minute, they receive identical sort keys. Since there's no secondary tiebreaker, the sort order between those tied files is not guaranteed to be consistent, so when the list re-sorts (toggling between Newest and Oldest, or after a new upload triggers a re-render), files with matching timestamps can swap positions relative to each other.
This explains every specific symptom from the original report:

Files uploaded minutes apart (outside the same hour:minute window) never had the problem. 


### Proposed Solution
Add a deterministic secondary sort key so files with identical hour:minute timestamps always resolve to the same, correct relative order:

Preferred: compare timestamps at full precision (down to seconds/milliseconds) if that data is available anywhere in the upload metadata, rather than truncating to hour:minute.
Fallback/supplement: if full-precision timestamps aren't available or reliable, use a stable secondary key, file id, or upload sequence/insertion order, as a tiebreaker in the comparator whenever primary timestamps are equal. This guarantees the sort is deterministic (same input always produces same output) regardless of how many files share a timestamp.
Either approach eliminates the swap-on-re-render behavior, since ties will no longer exist for the comparator to resolve inconsistently.


### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** Files uploaded within the same hour:minute window share an identical sort key. With no tiebreaker, the relative order between tied files is unstable across re-sorts/re-renders, causing them to visibly swap positions. 

**Match:** This is a standard "insufficient sort key precision / missing tiebreaker" bug. the same pattern behind many list-ordering bugs where multiple items share a coarse key. The fix (finer-grained comparison, or a stable secondary key) is a common pattern.


**Plan:** [Step-by-step implementation plan]
1. Locate the date/sort comparator for the Files & Uploads list (via grep -rn "dateAdded" src/files-and-videos/ and the sort-related files found from grep -rni "sortBy|sortType|sortable"), and confirm exactly where the hour:minute truncation happens. API response shape, a formatting/display step, or the comparator itself.
   
2. Check whether the underlying API already returns full-precision (seconds/ms) timestamps that are simply being truncated client-side before comparison, versus the truncation happening upstream (in which case a client-only secondary key is the right fix).

3. Write a failing unit test: seed multiple files with timestamps identical to the minute, assert that sorting Newest/Oldest produces a consistent, deterministic order across repeated sort calls.

4. Implement the fix, full-precision comparison if available, otherwise a stable secondary key (file id / insertion order) added to the comparator.
   
5.Re-run the manual reproduction (upload several files within the same minute, toggle Newest/ Oldest repeatedly) to confirm the order no longer changes between toggles.

6.Add the passing regression test alongside existing sort tests, and update/close out the maintainer comment on #3096 with this precise root cause and fix.

**Implement:** https://github.com/anshuanjna/frontend-app-authoring 

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

### Week [6] Progress
Understood the issue better and found a rough solution to follow through and now i have to implemenent the solution. 



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
