# SalesGenie AI — Sprint 1 Backlog

**Status:** Core implementation complete; Milestone 1 acceptance evidence and sign-off remain open (18 October 2026)
**Planning window:** 12–20 October 2026
**Prepared:** 15 October 2026
**Milestone goal:** Deliver a reliable lead creation-to-saved-company-insight flow
**Repository supplied:** `https://github.com/230701224/sales-genie-ai.git`

> The updated task rows assign kickoff/SRS to the Project Lead + Full Team, and API/stack/repository/environment planning to the Project Lead + Backend Developer + Database/DevOps Engineer. The backlog therefore assigns by role, not by individual name. The older milestone plan's individual mapping should not override the updated sheet without team confirmation.

## 1. Sprint outcome

A user can create or import a sample lead, retrieve/search/update it, request company analysis, and view the saved analysis against the same lead. The team can run the agreed local setup and demonstrate the flow. Test evidence includes the agreed test-set denominator and result.

## 2. Backlog items

| ID | Priority | Work item | Owner role(s) | Acceptance criteria | Status |
|---|---|---|---|---|---|
| SG-01 | P0 | Review scope and approve SRS | Project Lead + Full Team | Six modules and Milestone 1 scope are reviewed; decisions/open questions are recorded; sign-off table is completed by team members | SRS draft prepared; formal team review and signatures pending |
| SG-02 | P0 | Agree architecture, lead fields, and database model | Project Lead + Backend Developer + Database/DevOps Engineer | Shared lead/analysis fields, status values, foreign keys, SQLite/PostgreSQL behavior, and migration approach are approved | Eight-table schema implemented and validated on SQLite and PostgreSQL 16; requester reports draft status and duplicate policies approved; formal schema/contract sign-off pending |
| SG-03 | P0 | Finalize technology baseline | Project Lead + Backend Developer + Database/DevOps Engineer | Python/FastAPI/Streamlit/SQLAlchemy/Alembic, SQLite dev/PostgreSQL production, OpenAI provider, and exact model/version follow-ups are recorded | Selected stack recorded; requester selected OpenAI and `gpt-4o-mini`; live credentials and provider run remain unavailable |
| SG-04 | P0 | Approve API contracts | Backend Developer + Project Lead + Database/DevOps Engineer | Create/list/get/update/status/import/analysis contracts and common errors are agreed and captured in OpenAPI | Implemented draft is exposed through OpenAPI and covered by tests; formal contract review/sign-off pending |
| SG-05 | P0 | Establish repository and branching workflow | Project Lead + Backend Developer + Database/DevOps Engineer | Access to supplied remote is verified; `main`/`develop` policy and feature-branch/PR rules are agreed; secrets are excluded | Supplied remote was found to contain only a one-line README; workspace has no Git metadata; checkout/branch details and publication remain unresolved |
| SG-06 | P0 | Prepare repeatable local development and Docker baseline | Database/DevOps Engineer + Backend Developer | Setup works on a clean Windows machine; documented app ports, database config, sample data, and Docker/local DB workflow are validated | Setup instructions, local-safe `.env.example`, synthetic CSV, and live local API/UI startup verified; clean-machine team walkthrough pending |
| SG-07 | P1 | Implement lead schema, migration, and synthetic seed data | Database/DevOps Engineer + Backend Developer | Migration applies to the agreed databases; seed data can be loaded repeatably and contains no real customer data | Eight-table schema/migration and repeatable seed implemented; SQLite and ephemeral PostgreSQL 16 migrations/drift checks passed; three-row synthetic seed verified |
| SG-08 | P1 | Implement lead CRUD, search, status, and validation API | Backend Developer | Contracted endpoints pass create/read/update/search/status tests; invalid input and unknown IDs return explicit errors | Implemented; API tests cover CRUD/search/status, validation, duplicate rules, and missing IDs |
| SG-09 | P1 | Implement manual and CSV import with duplicate reporting | Backend Developer + Database/DevOps Engineer | Valid rows are created, duplicate/invalid rows are reported, and the outcome counts reconcile | Implemented with requester-selected all-or-nothing behavior; tests verify atomic rollback, row numbers, batch counts, and valid-file commit |
| SG-10 | P1 | Implement and persist company analysis | Backend Developer + AI/Analysis role | Analysis uses agreed inputs, returns the agreed shape, stores against the correct lead, and provider errors do not appear as success | OpenAI provider, saved insight, and explainable heuristic implemented; deterministic test provider verifies persistence/failure contract; live request attempted but provider returned HTTP 429; successful real-provider run and insight-quality review pending |
| SG-11 | P1 | Connect Leads screen to the API | Frontend role + Backend Developer | User can add, search, inspect, update status, analyze, and view saved insights | Leads, Add Lead, and CSV Import screens implemented; browser smoke verified list, create, field clearing, and atomic duplicate reporting; successful live-analysis display walkthrough pending |
| SG-12 | P1 | Run tests, record evidence, rehearse demo, and report risks | Project Lead + Full Team | Results report denominator and outcome against the sheet's >=95% manageable-record/insight-accuracy target; defects have owners/disposition; demo uses approved synthetic data | SQLite 13 passed/1 PostgreSQL-only skipped; PostgreSQL 16 14 passed; deterministic API flow 20/20; no live-model accuracy result or agreed quality rubric/test set; full demo, defect review, and sign-off remain open |

## 3. Suggested sequencing

1. Close SG-01 through SG-04 first; these define shared scope, data shape, and endpoint contracts.
2. SG-05 and SG-06 establish repeatable collaboration and local execution. Do not put populated `.env` files or API keys in Git.
3. SG-07 must be ready before database-backed CRUD and analysis integration.
4. SG-08 and SG-09 provide leads for SG-10/SG-11.
5. Complete SG-12 against the integrated flow and record evidence before submission.

The source milestone plan sets 19 October as demo/rehearsal and 20 October as submission/sign-off. The sheet's >=95% target is a Milestone 1 acceptance target. The recorded 20/20 deterministic API flow proves operation handling for those synthetic cases only; it does not measure real generated-insight accuracy. Backlog statuses above are limited to evidence produced in this workspace; they do not substitute for stakeholder review or signatures.

## 4. Risks and dependencies

- **Repository/source alignment:** the public remote currently contains only a one-line README, while the application implementation is in this non-Git workspace. Coordinate import/reconciliation before pushing or opening a PR.
- **Acceptance evidence outstanding:** the denominator and reviewer rubric for manageable records and accurate insights have not been agreed; the >=95% sheet target is therefore not yet demonstrated.
- **AI output quality:** configured-key live request returned HTTP 429 (specific quota/rate cause not exposed); no real model analysis or human accuracy review has been completed.
- **Human acceptance outstanding:** clean-machine setup, full demo rehearsal, defect disposition, SRS/API review, and team signatures remain team-owned.
- **Repository/source alignment:** reconcile this workspace with the remote only after the correct checkout/branch workflow is supplied; do not overwrite or push unreviewed work.
- **Provider guidance differs:** some supplied execution text mentions Groq while the selected project baseline is OpenAI; retain the decision record and resolve any guide conflict before submission.
- **Data/privacy:** use synthetic sample records in demos and tests; protect keys and any real prospect data.
