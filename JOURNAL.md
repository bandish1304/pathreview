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

## Week 8 - Local reproduction

What I tested:
I ran the app locally, logged in with a seeded test account, created one profile with the same GitHub username, portfolio URL, and resume, and then submitted two review requests for that exact same profile.

What I expected:
If caching were already in place, the second request should have reused the first completed review because nothing about the profile changed.

What actually happened:
The app created two different review records for the same unchanged profile, and both of them finished processing. That shows the current review flow does not check for an existing cached result before creating and processing a new review.

Where the issue appears to live:
The main gap is in the review creation and processing flow in core/services/review_service.py. Right now the service always creates a new review and runs the processing path again, and there is no content-hash lookup to short-circuit duplicate requests.
