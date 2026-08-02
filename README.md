# CodePath301

# Contribution [#3096]: [Bug]: [Test Failure] TC-00141: Files section appears to sort newest/oldest in the wrong order

**Contribution Number:** #3096
**Student:** Anshu Anjna
**Issue:** https://github.com/openedx/frontend-app-authoring/issues/3096
**Status:** Phase 3 - Completed

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
There are actually 2 bugs, one small, and one big. The small one is that the upload timestamp is only stored and compared at hour:minute, not down to the second. When two or more files are uploaded within the same minute, they receive identical sort keys. Since there's no secondary tiebreaker, the sort order between those tied files is not guaranteed to be consistent, so when the list re-sorts (toggling between Newest and Oldest, or after a new upload triggers a re-render), files with matching timestamps can swap positions relative to each other.

This big problem is the one as stated above from the maintainer: "Sorting by Newest places the most recently uploaded file at the bottom of the list and sorting by Oldest places the most recently uploaded file at the top of the list." I had thought this wasn't the issue because when I had first reproduced the issue, this never came up, but when i rechecked the web-app(after my first update to the maintainer), and uploaded new files to check, this bug became visible to me.


### Proposed Solution
For the small bug: Add a deterministic secondary sort key so files with identical hour:minute timestamps always resolve to the same, correct relative order:

Preferred: compare timestamps at full precision (down to seconds/milliseconds) if that data is available anywhere in the upload metadata, rather than truncating to hour:minute.
Fallback/supplement: if full-precision timestamps aren't available or reliable, use a stable secondary key, file id, or upload sequence/insertion order, as a tiebreaker in the comparator whenever primary timestamps are equal. This guarantees the sort is deterministic (same input always produces same output) regardless of how many files share a timestamp.
Either approach eliminates the swap-on-re-render behavior, since ties will no longer exist for the comparator to resolve inconsistently.

For the big bug: Stop converting the timestamp to a formatted string before comparing it. Replace new Date(utcDateString).toString() with new Date(utcDateString).getTime(), which returns a plain numeric epoch timestamp with no weekday text or other formatting baked into it. Numeric comparison (1691964480000 < 1691964490000) is always chronologically correct regardless of what day, month, or year the files fall on — there's nothing left in the value for a string comparison to misread.


### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** Files uploaded within the same hour:minute window share an identical sort key. With no tiebreaker, the relative order between tied files is unstable across re-sorts/re-renders, causing them to visibly swap positions. Also for the bigger bug: the date value used for sorting was being rendered out to a human-readable string (Date.prototype.toString()) before comparison, and that string happens to start with a day-of-week abbreviation. Comparing those strings directly meant the "sort" was really alphabetizing by weekday name in disguise, which is unrelated to and independent of actual chronological order — a genuinely incorrect sort, not just an unstable one.

**Match:** This is a standard "insufficient sort key precision / missing tiebreaker" bug. the same pattern behind many list-ordering bugs where multiple items share a coarse key. The fix (finer-grained comparison, or a stable secondary key) is a common pattern. The big bug is a different, equally common pattern: "sorting on a human-readable string representation instead of a directly comparable value" — the classic trap of assuming a formatted display string is safe to also use for comparison logic.


**Plan:** [Step-by-step implementation plan]
1. Locate the date/sort comparator for the Files & Uploads list (via grep -rn "dateAdded" src/files-and-videos/ and the sort-related files found from grep -rni "sortBy|sortType|sortable"), and confirm exactly where the hour:minute truncation happens. API response shape, a formatting/display step, or the comparator itself.
   
2. Check whether the underlying API already returns full-precision (seconds/ms) timestamps that are simply being truncated client-side before comparison, versus the truncation happening upstream (in which case a client-only secondary key is the right fix).

3. Write a failing unit test: seed multiple files with timestamps identical to the minute, assert that sorting Newest/Oldest produces a consistent, deterministic order across repeated sort calls.
   
5.Re-run the manual reproduction (upload several files within the same minute, toggle Newest/ Oldest repeatedly) to confirm the order no longer changes between toggles.

6.Add the passing regression test alongside existing sort tests

**Implement:** https://github.com/anshuanjna/frontend-app-authoring 

**Review:** 
Self-reviewed both fixes primarily through manual testing rather than assuming passing unit tests meant the work was done. This caught a real mistake: my first tiebreaker implementation passed its own unit test but was still wrong in practice — it kept tied files (file8, file9) frozen in the same order regardless of sort direction, which only became visible when manually toggling Newest/Oldest with a real same-second upload pair. Also caught and corrected a stale code comment that claimed the tiebreaker was "intentionally NOT multiplied by directionMultiplier" after the surrounding code had already been changed to do exactly that, a reminder to re-read comments against the actual code during self-review, not just trust that they're still accurate. 


**Evaluate:** The fix resolves both reported symptoms from the original issue, and this is now backed by three layers of testing, not just one: same-minute uploads no longer scramble unpredictably (they now flip consistently and correctly along with the sort direction, same as any other file), and files spanning different weekdays now sort in genuinely correct chronological order instead of alphabetical-by-weekday order — verified via unit tests on the underlying functions, an integration test on the actual rendered UI, and manual browser testing with real uploaded files. Honest limitations still worth stating rather than glossing over: (1) the id-based tiebreaker guarantees deterministic, direction-consistent ordering for same-minute files, but does not guarantee that ordering matches true upload sequence, that would require backend-level sub-minute timestamp precision, out of scope here; (2) the integration test covers the default gallery/card view only, the alternate list/table view (DataTable.Table) hasn't been separately verified to render the same order, which would be worth a quick manual check or a follow-up test case before considering this fully complete across the whole UI.

---

## Testing Strategy

### Unit Tests

- [X] Test case 1: Same-minute uploads produce a consistent relative order across both Newest and Oldest sort directions (tiebreaker regression test).
- [X] Test case 2: Files with distinct timestamps sort correctly regardless of which weekday they fall on (chronological/.getTime() regression test).
- [X] Test case 3: updateFileValues converts dateAdded into a numeric epoch timestamp (not a string), and two files uploaded in the same minute produce equal dateAdded values (confirms .getTime() correctly identifies real ties without introducing false ones — files-page/data/utils.test.js).

### Integration Tests

- [X] Integration scenario 1 - Same-minute tied files, rendered via the real FilesPage component and a real Sort & Filter modal interaction, produce a consistent, direction-respecting order across repeated Newest/Oldest toggles (FilesPage.sort.test.jsx).
- [X] Integration scenario 2 - Files spanning different weekdays, rendered via the real FilesPage component, sort in genuinely correct chronological order in both directions (FilesPage.sort.test.jsx).

### Manual Testing

Reproduced the original tie-scrambling bug in devstack (Root Cause #1): uploaded files within the same minute, confirmed their order changed unpredictably when toggling Newest/Oldest repeatedly.
Reproduced the weekday-string bug (Root Cause #2): uploaded new files (file5, file6) on a different day than earlier test files, confirmed they sorted as "oldest" despite being the most recently uploaded.
Verified Root Cause #2 was a separate, pre-existing bug (not something my in-progress tiebreaker fix introduced) by reverting that fix and confirming the weekday issue persisted on its own.
Applied the .getTime() fix and confirmed correct chronological ordering afterward.
Uploaded two more files seconds apart (file8, file9) to specifically stress-test the tiebreaker: found that my first tiebreaker implementation kept the tied pair in the same relative order regardless of sort direction (e.g. file8 always above file9, even under Newest, where file9 should have been on top). Confirmed via repeated toggling (Newest → Oldest → Newest, several times) that this was a consistent wrong answer, not a return of the original random-scrambling bug — which correctly narrowed it down to the tiebreaker not respecting directionMultiplier, rather than a regression of Root Cause #1.
After correcting the tiebreaker to multiply by directionMultiplier, re-tested file8/file9 and confirmed they now flip correctly along with the sort direction, and stay consistent across repeated toggling and page reloads.

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

### Week [7] Progress
Stil working on solving the issue.

### Week [8] Progress
I'm still working on the solution. I have found where to change and edit the code, but I was traveling last week so couldn't work on the assignment. 

### Week [9] Progress
Implemented the tiebreaker fix in generic/utils.js: replaced the .reverse()-based ascending sort with an independent comparator per direction, and added file.id as a secondary sort key so tied (same-minute) files stay in a consistent relative order across toggles. 

While manually re-testing this fix with newly uploaded files, discovered the original bug: files uploaded on certain weekdays sorted out of chronological order relative to older files, even with the tiebreaker fix in place and even after reverting it, proving it was a separate, pre-existing issue. Traced this to files-page/data/utils.js converting timestamps with new Date(x).toString(), which puts the day-of-week abbreviation at the front of the string and causes alphabetical (not chronological) comparison. 

Fixed by switching to new Date(x).getTime(), which sorts on a plain numeric timestamp instead. Applied this fix to the Files page.



### Code Changes

- **Files modified:**
Files modified (4 total: 2 source files, 2 new test files):

src/files-and-videos/generic/utils.js — rewrote the sortFiles comparator: replaced the .reverse()-based ascending sort with an independent, direction-aware comparator (directionMultiplier), added an id-based secondary tiebreaker for same-timestamp files, and applied directionMultiplier to that tiebreaker as well so tied files flip consistently with the rest of the list.

1. src/files-and-videos/files-page/data/utils.js — changed the dateAdded conversion in updateFileValues from new Date(utcDateString).toString() to new Date(utcDateString).getTime(), fixing the weekday-string comparison bug.

2. src/files-and-videos/generic/utils.test.js (new file) — regression test for sortFiles, covering both non-tied files (correct chronological order) and tied files (consistent, direction-respecting order across Newest/Oldest toggles).

3. src/files-and-videos/files-page/data/utils.test.js (new file) — regression tests for updateFileValues, covering: dateAdded is converted to a number, the converted value matches a real getTime() parse, files on different weekdays sort chronologically correctly, and same-minute files produce equal (correctly tied) values.

4. src/files-and-videos/files-page/data/utils.js — changed date conversion from .toString() to .getTime() (chronological fix)
5. src/files-and-videos/files-page/FilesPage.sort.test.jsx (new file, integration test) — renders the real FilesPage component (not just the sort function in isolation) with a 4-file fixture combining both bug scenarios (a same-minute tied pair, and a genuinely-older/newer pair spanning weekdays whose abbreviations sort "backwards"), clicks through the real Sort & Filter modal exactly as a user would, and asserts the actual rendered card order in the DOM. Confirms: the newer-weekday file renders before the older one under Newest (and reversed under Oldest); the tied pair's order flips consistently between directions; and toggling back to Newest a second time reproduces the same tied order as the first time. Passed on first real run against the fixed code (PASS src/files-and-videos/files-page/FilesPage.sort.test.jsx, 1 test, 1.77s).

- **Key commits:** [Links to important commits]
- **Approach decisions:**
Chose .getTime() over .toISOString() for the chronological fix. Both would have technically fixed the weekday-string bug (ISO strings sort correctly as text), but .getTime() returns a plain number, which removes string formatting from the comparison entirely rather than trading one string format for another — a smaller, more unambiguous surface area to reason about, and it can't be broken again by a future formatting change upstream.

Chose file.id over an upload-sequence counter for the tiebreaker, since no reliable sequence field is currently exposed to the frontend by the API. Documented this as a known limitation: the tiebreaker guarantees deterministic, direction-consistent ordering for tied files, but does not guarantee the order matches true upload sequence for files uploaded in the same minute, that would require a backend change (e.g. a real sequence number or sub-minute timestamp), which is out of scope for this frontend fix and worth raising as a possible follow-up issue.

Applied directionMultiplier to the tiebreaker rather than leaving it direction-independent (my first attempt). Initially reasoned that multiplying it would "reintroduce a milder version of the original bug," but that reasoning didn't hold up: Array.prototype.sort() always does a well-defined pairwise comparison, tied or not — there's no risk of the random scrambling the original .reverse()-based code caused. Manual testing with file8/file9 (uploaded seconds apart) surfaced the mistake directly: without the multiplier, tied files stayed in a fixed order regardless of direction, which looked wrong and inconsistent with how every other file in the list behaved when toggled.

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** Awaiting review 

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]
I learned to distinguish between two superficially similar but independently-caused sorting bugs by isolating variables (reverting one fix to confirm the other issue was pre-existing rather than newly introduced). Learned the internals of Date.prototype.toString() vs .getTime() vs .toISOString() and why string-comparing formatted dates is fragile. Practiced tracing a frontend field (dateAdded/created) back through the API layer to a backend serializer to reason about data format rather than guessing.

### Challenges Overcome

[What was hard and how you solved it]
The second bug (weekday-string comparison) was easy to miss because it's not visible with a small, tightly-clustered set of test files — it only appears once uploads span multiple weekdays whose abbreviations sort "out of order" alphabetically. Diagnosing it required building a concrete counter-example rather than relying on visual inspection alone.

### What I'd Do Differently Next Time

[Reflection on your process]
I would test sorting behavior against a wider spread of dates (not just files uploaded minutes apart) earlier in the process, since the weekday-string bug would have been caught sooner with test data spanning multiple weeks/weekdays from the start.

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
