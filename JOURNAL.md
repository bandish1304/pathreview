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
