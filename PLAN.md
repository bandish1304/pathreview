## Solution plan

Issue: Implement a caching layer for repeated identical portfolio queries
https://github.com/ascherj/pathreview/issues/32

### Understand

The expected behavior is that if a user requests a review for the same unchanged profile twice, the second request should reuse the earlier completed review instead of doing the same work again. The actual behavior is that the app creates a second review record and processes it as if it were a brand new request.

From the reproduction work, the root cause looks like a gap in the review creation flow. The service creates a new review every time, and the route immediately schedules background processing after that. There is no stored review-level fingerprint for the profile input and no cache lookup before the new review is inserted.

### Map

The main files and modules involved are:

- core/services/review_service.py
- api/routes/reviews.py
- core/models/review.py
- alembic/versions/
- tests/unit/test_review_service.py

The key behavior lives in create_review and process_review inside core/services/review_service.py, plus the POST /reviews route in api/routes/reviews.py.

### Plan

1. Add a deterministic content hash for the review input based on the profile fields that affect generated output, especially GitHub username, portfolio URL, and resume text.
2. Persist that hash on the review record so completed reviews can be matched against future identical requests.
3. Update review creation to load the profile, compute the current hash, and look for an existing completed review with the same content before inserting a new one.
4. Only create a new pending review and schedule background processing when there is no cache hit.
5. Add or update unit tests to cover both the cache-hit path and the cache-miss path.

### Inputs and outputs

Input:
The fix takes the current profile content used to generate a review, mainly GitHub username, portfolio URL, and resume text.

Output:
If the content matches a previously completed review, the app should return that existing review instead of creating a duplicate one. If the content changed, the app should create a new review and process it normally.

### Risks and unknowns

- The issue description talks about the RAG pipeline, but parts of the current local implementation are still placeholder logic, so I need to make sure the cache entry point still makes sense for the intended full pipeline.
- There is already a content_hash field on ingested sources, but that is not enough to stop duplicate review creation at request time.
- During reproduction I also saw a separate ingestion error involving raw_data on IngestedSource. That looks unrelated, but I need to make sure it does not interfere with validation.

### Edge cases

- A user submits the same profile twice with no changes at all.
- A user changes only one field, such as the resume text or portfolio URL, and should get a fresh review.
- A previous matching review exists but is failed or still processing, which should probably not count as a cache hit.
- Empty or missing optional fields should still produce a stable hash and should not crash the lookup path.