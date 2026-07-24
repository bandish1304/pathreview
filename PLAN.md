# Solution Plan - Issue 32

## Issue

Implement a caching layer for repeated identical portfolio queries.

## What I reproduced locally

I created one profile with the same GitHub username, portfolio URL, and resume, then requested two reviews for that exact same profile. Both requests created separate review records and both reviews completed successfully. That confirms the app does not currently reuse an earlier result when the input has not changed.

## Problem in plain language

Right now the review flow treats every request like a brand new job, even when the profile content is identical to a previous submission. That means the app does duplicate work and stores duplicate review results for the same unchanged input. A successful fix should detect that nothing changed, find the matching completed review, and return that result instead of starting a new review run.

## Root cause hypothesis

The main gap is in the review creation flow in core/services/review_service.py. The create_review function always inserts a new review, and the request path in api/routes/reviews.py always schedules background processing immediately after that. There is no profile content fingerprint, no cache lookup before review creation, and no stored link between a completed review and the exact input that produced it.

## Proposed approach

1. Add a deterministic content hash for review inputs.

I want to hash the parts of the profile that affect review output: GitHub username, portfolio URL, and resume text. The hash should be built from normalized values so small formatting differences do not create unnecessary cache misses.

2. Persist that hash with each review.

The current Review model does not store a content hash, so there is nowhere reliable to look up a previously completed result before starting work. I plan to add a new column on reviews for the content hash and generate an Alembic migration for it.

3. Check for a cached completed review before creating a new one.

Before inserting a new review, the service should load the profile, compute the current hash, and search for an existing completed review for the same profile content. If one exists, the API should return that review instead of creating a duplicate pending review.

4. Only start background processing on cache misses.

If no matching completed review exists, the app should create a new pending review, store the computed hash on it, and continue with the existing processing flow.

5. Add tests around both paths.

I want one test that proves a repeated identical request reuses the existing completed review, and another that proves changed profile content produces a new review. I also want to keep coverage for the current new-review path so the change does not break normal review creation.

## Files I expect to touch

- core/services/review_service.py
  This is where the review creation decision happens now, so it is the main place for hash generation, cache lookup, and reuse logic.

- api/routes/reviews.py
  The route may need a small adjustment so it only schedules background processing when the request actually created a new pending review.

- core/models/review.py
  I likely need to add a persisted content_hash field so completed reviews can be matched against future identical requests.

- alembic/versions/
  I expect to add a migration for the new review content hash column and any supporting index.

- tests/unit/test_review_service.py
  This file already covers create_review behavior, so it is the natural place to add cache-hit and cache-miss tests.

## Validation plan

1. Re-run the local reproduction flow with the same unchanged profile and confirm the second request returns the existing review instead of creating a new one.
2. Change one input field, such as the resume text or portfolio URL, and confirm that a new review is created.
3. Run the unit tests for review service behavior.
4. If needed, do one quick end-to-end API check for login, profile creation, first review, and repeated review.

## Risks and unknowns

- The current issue mentions the RAG pipeline, but this local branch uses placeholder review generation in parts of the flow. I need to make sure the cache design still fits the intended production path.
- There is already a content_hash field on ingested sources, but that is too late in the flow to prevent duplicate review creation. I need to be careful not to confuse source-level deduplication with review-level caching.
- I also saw a separate ingestion error involving raw_data on IngestedSource during reproduction. That looks unrelated to the cache issue, so I plan to avoid pulling that into this fix unless it directly blocks validation.

## Definition of done

The fix is done when a repeated review request for unchanged profile content returns the previously completed review, no duplicate pending review is created for the same content, and a changed profile still triggers a new review as expected.