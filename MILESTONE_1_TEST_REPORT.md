# SalesGenie AI — Milestone 1 Verification Report

**Verification date:** 18 October 2026
**Status:** Core implementation verified; Milestone 1 acceptance is not yet complete.

## Scope exercised

- Lead creation, retrieval, paginated search, detail update, lifecycle update, validation, duplicate handling, and missing-ID errors.
- CSV all-or-nothing import, duplicate/invalid row reporting, and stored import counts.
- Company analysis response, lead linkage, saved insight retrieval, qualification persistence, and provider-configuration failure behavior.
- SQLite schema creation, normalized duplicate indexes, and lead-delete cascade behavior.
- Synthetic 20-prospect API flow for create, retrieve, update, status update, analyze, and retrieve saved analysis.
- Browser flow for adding a lead, successful form clearing, and atomic duplicate CSV rejection.

## Results

| Evidence | Result |
|---|---|
| SQLite API/DB automated tests | 13 passed, 1 PostgreSQL-only test skipped |
| Synthetic API flow | 20 / 20 records completed through deterministic test-provider flow; this tests operation handling, not real AI insight accuracy |
| Test database | Temporary SQLite database created from SQLAlchemy metadata for API tests |
| SQLite migration database | `alembic upgrade head`, `alembic check`, and `alembic current` succeeded |
| PostgreSQL migration database | PostgreSQL 16 in an ephemeral Docker container: upgrade, drift check, API integration tests (14 passed), and downgrade succeeded |
| Synthetic seed | 3 rows inserted into the local SQLite database |
| Live API/UI startup | API health and OpenAPI served successfully; Streamlit health endpoint returned `ok` |
| Browser smoke | Leads screen rendered 3 seeded leads; manual creation and profile/status updates succeeded; Add Lead cleared all fields after success; sample CSV was rejected atomically with 3 duplicates, 0 invalid rows, and 0 valid rows rolled back (no rows imported) |
| AI provider in automated flow | Deterministic test-only provider; no external API call |
| Live OpenAI attempt | Request reached the configured provider but returned HTTP 429 (`RateLimitError`), surfaced by the API as `AI_PROVIDER_ERROR`; no analysis result was returned. The error response did not reveal whether the limit is quota/billing or request rate. Synthetic lead and temporary database were cleaned up. |
| Data used | Synthetic records only |

Run the SQLite test suite with:

```powershell
python -m pytest db\tests\test_api.py db\tests\test_schema.py -q
```

The PostgreSQL CI job runs `python -m pytest db/tests -q` against PostgreSQL 16; all 14 tests passed in the local PostgreSQL 16 verification. The PostgreSQL-specific integration test uses a test-only analysis provider and deletes the unique synthetic rows it creates.

## Limitations and acceptance work still open

- The sheet's >=95% Milestone 1 target is not yet demonstrated. The team has not supplied an agreed denominator/test set or rubric for judging insight accuracy. The 20/20 result is an API/database flow with a deterministic test provider; it is not a real-model accuracy result or a substitute for insight review.
- PostgreSQL schema, migration parity, and API integration flow were verified in PostgreSQL 16. A clean-machine Docker Compose setup walk-through remains for the team.
- Full Streamlit acceptance walkthrough for viewing a successful saved analysis remains. A live request was attempted with the configured local key, but OpenAI returned HTTP 429; no saved result was produced. Check the OpenAI project's API billing/usage limits or rate limit, then retry the live analysis and saved-insight retrieval.
- The requester reports team approval of the selected provider and draft lifecycle/duplicate policies. The >=95% evaluation denominator and insight-quality rubric, formal requirements/API sign-off, defect disposition, demo rehearsal, and SRS signatures remain open.
- The supplied GitHub repository currently contains only a one-line README. This workspace is not a Git checkout, so these changes have not been pushed or submitted as a pull request.

SQLite cannot reflect the expression-based normalized-email indexes for Alembic autogenerate comparison. PostgreSQL 16 `alembic check` passed with the normalized indexes semantically compared; duplicate behavior is separately covered by migration and API tests.
