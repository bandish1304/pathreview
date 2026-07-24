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
