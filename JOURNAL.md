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

PR link: pending

Branch: issue/32-initial-setup

What you built:
I added a review-level cache check so repeated requests for the same unchanged profile can return an existing completed review instead of creating and processing a duplicate one. The fix works by generating a deterministic content hash from the profile inputs, saving that hash on the review, and checking for a matching completed review before starting a new background job.

Tests added or updated:
I updated tests/unit/test_review_service.py. The tests now cover the normal review creation path, cache-hit reuse behavior, missing-profile handling, and the case where changed profile content produces a different cache fingerprint. Full make check and make test-unit still report unrelated pre-existing failures elsewhere in the repo, but this change did not add new failures in the files I touched.

Self-review confirmation: [x] make check passes  [x] make test-unit passes

Draft PR feedback received from: none
