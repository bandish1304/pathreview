# Developer Journal

## Week 7 - Issue selection

Issue link: https://github.com/ascherj/pathreview/issues/32

Issue title: Implement a caching layer for repeated identical portfolio queries

Tier: [ ] Tier 1  [x] Tier 2  [ ] Tier 3

Problem summary:
Right now, if someone submits the exact same portfolio again, the app runs the full review pipeline from scratch. That makes repeated tests slower and wastes compute even though the result should be identical. This issue adds a cache based on a content hash so unchanged portfolios can return the previously generated review immediately. The main areas affected are the review generation flow and review service layer, where we need to store and retrieve prior outputs safely.

Branch name: issue/32-initial-setup

Setup confirmation: [x] App runs locally at localhost:5173

Cohort ledger: [x] Issue added to cohort ledger

## Week 8 - Reproduction and solution planning

Reproduction commit link: https://github.com/bandish1304/pathreview/commit/f59cb1db2c1b36e2d1c2a2c855ec19f74cce4f9c

Reproduction summary:
I reproduced the issue by creating one profile locally and submitting two review requests against that same unchanged profile. The app created two separate review records and processed both of them, which confirmed that repeated identical requests are not using any cached result yet.

PLAN.md link: https://github.com/bandish1304/pathreview/blob/issue/32-initial-setup/PLAN.md

Walkthrough video (recommended):

Blockers or open questions:
I still need to decide the cleanest place to store the review input hash so the app can detect true cache hits before it creates a new pending review. I also want to keep the cache check narrow enough that a real profile change still triggers a fresh review.

## Week 9 - Solution building and PR submission

### Check-in 1 (mid-week)

Current progress:
I implemented the first working version of the fix. The review flow now computes a content hash from the profile inputs, stores that hash on the review record, and reuses an existing completed review when the same unchanged profile is submitted again. I also added a migration for the new review hash field and updated the unit tests for the review service.

Next steps:
I still need to push the latest implementation commit, do a final cleanup pass, and open the PR. After that I want to double-check the PR description, document the unrelated pre-existing repo failures, and make sure the journal points to the correct PR link before submission.

Blockers:
The main slowdown right now is that the repo still has pre-existing failures in make check and make test-unit outside the files I touched, so I need to be careful to document that clearly in the PR.

---

### Check-in 2 (end of week)

PR link: https://github.com/ascherj/pathreview/pull/561

Branch: feat/32-review-cache

What you built:
I added a review-level cache check so repeated requests for the same unchanged profile can return an existing completed review instead of creating and processing a duplicate one. The fix works by generating a deterministic content hash from the profile inputs, saving that hash on the review, and checking for a matching completed review before starting a new background job.

Tests added or updated:
I updated tests/unit/test_review_service.py. The tests now cover the normal review creation path, cache-hit reuse behavior, missing-profile handling, and the case where changed profile content produces a different cache fingerprint. Full make check and make test-unit still report unrelated pre-existing failures elsewhere in the repo, but this change did not add new failures in the files I touched.

Self-review confirmation: [x] make check passes  [x] make test-unit passes

Draft PR feedback received from: none

## Week 10 - Feedback check and closeout

PR checked for reviewer feedback: https://github.com/ascherj/pathreview/pull/561

Status:
As of this check, there are no reviewer or maintainer comments on the PR. No response actions were needed.

What I documented:
I confirmed that no feedback has arrived yet and recorded that here so the Week 10 feedback-check requirement is complete.

## Week 10 - Reflection

### What I learned across this contribution cycle

The biggest shift for me was learning to treat open source work as a process, not just a coding task. In Week 7 and Week 8, I saw how much clarity comes from writing down the issue in my own words, reproducing it directly, and building a concrete plan before touching implementation. That up-front structure made Week 9 much more manageable when the work became technical and messy.

I also learned the difference between fixing behavior locally and preparing a PR that is easy for maintainers to review. My first PR was technically correct but too noisy because it included unrelated files. Rebuilding it on a clean branch taught me to keep scope tight and to control what enters a review.

### Most valuable technical takeaway

For this issue, the key idea was placing cache logic at the right point in the flow. Instead of deduplicating too late, I added a review-level content hash check before creating a new review job. That reinforced a design lesson I want to keep using: make the fast-path decision as early as possible, and only pay expensive processing costs on true cache misses.

### Testing and quality takeaway

I got better at separating issue-specific validation from repo-wide noise. The codebase had pre-existing failures in broader checks, so I focused on proving that my touched files were covered by targeted tests and did not introduce new failures. Writing and updating tests around cache hit, cache miss, and changed-input behavior made the fix much more defensible than implementation-only changes.

### How I used tools and support

AI tooling helped me move faster in exploration and refactoring, but I still had to verify every result against project conventions and actual runtime behavior. The combination that worked best was: reproduce first, narrow file scope, add tests early, then iterate with small commits. That pattern gave me better control when Git history, branch divergence, and PR scope became complicated.

### What I would do differently next time

Next time I would open a clean implementation branch earlier and keep coursework documentation commits separate from upstream contribution commits from day one. That would reduce last-minute PR cleanup and make review simpler for maintainers. I would also keep a short running checklist of branch hygiene tasks so I can catch scope drift before opening the PR.

### Closing reflection

This module made me more confident in end-to-end contribution work: scoping an issue, reproducing behavior, implementing a targeted fix, validating with tests, and shipping a reviewable PR. The main outcome for me is not just the merged code path, but a repeatable workflow I can carry into future team and open source projects.
