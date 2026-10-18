# SalesGenie AI — Development and Repository Setup

**Status:** Core Milestone 1 application implemented; SQLite/PostgreSQL automated validation passed; live-provider and team acceptance remain open
**Prepared:** 18 October 2026
**Selected baseline:** Python + FastAPI + Streamlit + SQLAlchemy + Alembic; SQLite for local development, PostgreSQL for production; OpenAI API

> The supplied GitHub repository currently contains only a one-line README and has no application source. The implementation in this workspace is therefore the working source baseline. This document does not claim that changes have been pushed to GitHub or that all team acceptance/sign-off steps are complete.

## 1. Prerequisites

- Python 3.10 or newer (training-guide minimum); Python 3.11 is used by CI. This workspace was verified with Python 3.13.5.
- Git.
- Docker Desktop when running PostgreSQL locally.
- An OpenAI API key to run real company analysis. Obtain it through the team's approved secret-sharing process; never put a real key in Git or in shared documentation.

## 2. Repository and branch workflow

The supplied repository was inspected; its only file is `README.md` (`# sales-genie-ai`). The current project workspace is not a Git checkout. Before publishing this implementation, the project lead should coordinate with the repository owner, import/reconcile this source baseline, and agree on the branch/PR policy. Do not initialize or push a repository without that coordination.

```powershell
git clone https://github.com/230701224/sales-genie-ai.git
Set-Location .\sales-genie-ai
git fetch origin
git branch --all
```

Proposed branch policy:

- `main`: reviewed, demoable baseline; protect it from direct pushes.
- `develop`: shared integration branch, if the team agrees to use it.
- `feature/<ticket>-<short-name>`: short-lived task branches.
- Submit changes through pull requests; require at least one teammate review and passing tests before merging.
- Do not commit `.env`, local databases containing real data, virtual environments, caches, or credentials.

Before creating `develop`, check `git branch --all` and coordinate with repository owners to avoid creating a conflicting branch or overwriting existing work.

## 3. Windows virtual environment

From the current project root in PowerShell:

```powershell
python --version
git --version
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

If PowerShell blocks activation, use the team's approved execution policy or invoke the environment's Python directly:

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

The repository provides a root `requirements.txt`, `.env.example`, and `.gitignore`. Version ranges are declared in the manifest; the team should lock and approve exact versions before deployment.

## 4. Environment configuration

Copy the example file locally and fill only local values:

```powershell
Copy-Item .env.example .env
```

The checked-in `.env.example` contains local-safe placeholders:

```dotenv
DATABASE_URL=sqlite:///./salesgenie.db
OPENAI_API_KEY=
OPENAI_MODEL=gpt-4o-mini
API_BASE_URL=http://127.0.0.1:8000
```

Copy `.env.example` to `.env` and set a valid `OPENAI_API_KEY` for real analysis. `OPENAI_MODEL` is configurable; the requester reports `gpt-4o-mini` as the team-selected model. Use a PostgreSQL `DATABASE_URL` for production configuration and local parity tests when required. Keep actual passwords and API keys out of `.env.example`, documentation, terminal recordings, screenshots, and commits.

## 5. Local services

The existing guides/screenshots indicate a FastAPI backend on port `8000` and a Streamlit UI on port `8501`. Apply the migration and load repeatable synthetic data first:

```powershell
python -m alembic upgrade head
python -m db.seed
```

Start the API:

```powershell
python -m uvicorn main:app --reload --host 127.0.0.1 --port 8000
```

In a second PowerShell window, activate the same virtual environment and run:

```powershell
python -m streamlit run app.py --server.address 127.0.0.1 --server.port 8501
```

Expected local endpoints:

- API: `http://127.0.0.1:8000`
- API documentation: `http://127.0.0.1:8000/docs`
- Streamlit: `http://127.0.0.1:8501`

FastAPI serves interactive API docs at `http://127.0.0.1:8000/docs` and OpenAPI JSON at `http://127.0.0.1:8000/openapi.json`. The Streamlit Leads screen supports searching/filtering, lead detail edits and lifecycle status, company analysis, and viewing saved insights. Separate pages handle manual add and CSV import. CSV is all-or-nothing: any invalid or duplicate row rejects the file without creating any of its otherwise-valid rows.

Analysis requests require an OpenAI key and a configured model. With no key, the API returns an explicit provider-configuration error and does not create a success-shaped analysis record. The API and tests do not use mock success output; automated flow tests inject a deterministic test-only provider.

## 6. Database and Docker baseline

- Local development default: SQLite, selected from the training-guide baseline.
- Production target: PostgreSQL.
- Use SQLAlchemy for persistence and Alembic migrations for schema changes.
- The workspace includes an eight-table SQLAlchemy model and Alembic migration in `db/` and `alembic/`. The eight tables are documented in `db/README.md`. The requester reports team approval of the selected status values and duplicate matching rule; formal SRS/API sign-off remains open.
- `compose.yaml` runs PostgreSQL 16 for parity testing, binds its port to localhost, obtains its password from the ignored local `.env`, and persists data in a named volume. Never use the example password outside an isolated local environment.
- GitHub Actions is configured to validate upgrade/check/downgrade on SQLite and PostgreSQL. SQLite tests and PostgreSQL 16 migration/API tests passed locally; a remote GitHub Actions run is not recorded.

For PostgreSQL local development, copy `.env.example` to `.env`, replace the local-only password, and follow the tested command sequence in `db/README.md`. Never commit `.env`.

Docker Compose configuration is present. PostgreSQL 16 was validated in an isolated temporary container; the persistent Compose volume was not created or changed by that check.

## 7. Initial verification checklist

Verified in this workspace:

- `python -m pytest db\tests -q` passes 13 tests with one PostgreSQL-only test skipped when `TEST_DATABASE_URL` is unset. Against PostgreSQL 16, all 14 tests passed, including a 20-prospect API flow using a test-only provider.
- Tests use synthetic records and a temporary SQLite database; no provider key is needed for test execution.

Still required for Milestone 1 acceptance: clean-machine installation by the team; configure the key locally and run/retrieve/review a real OpenAI analysis; agree the test-set denominator and insight-accuracy rubric for the sheet's >=95% target; demo rehearsal; defect disposition; and formal team sign-off. PostgreSQL migration/API parity has been verified in an ephemeral PostgreSQL 16 container; see [MILESTONE_1_TEST_REPORT.md](./MILESTONE_1_TEST_REPORT.md).
