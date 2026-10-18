# SalesGenie AI

Milestone 1 implementation of the lead management and company intelligence flow.

## Start locally

1. Create a Python 3.10+ virtual environment and install `requirements.txt`.
2. Copy `.env.example` to `.env`; set `OPENAI_API_KEY` for live analysis.
3. Apply the schema and load synthetic records:

   ```powershell
   python -m alembic upgrade head
   python -m db.seed
   ```

4. In one terminal, start the API:

   ```powershell
   python -m uvicorn main:app --reload --host 127.0.0.1 --port 8000
   ```

5. In another terminal, start the UI:

   ```powershell
   python -m streamlit run app.py --server.address 127.0.0.1 --server.port 8501
   ```

Open the Streamlit app at `http://127.0.0.1:8501`; interactive API documentation is at `http://127.0.0.1:8000/docs`.

See [DEVELOPMENT_SETUP.md](./DEVELOPMENT_SETUP.md) for Windows, environment, and PostgreSQL instructions; [API_CONTRACT.md](./API_CONTRACT.md) for endpoints; and [MILESTONE_1_TEST_REPORT.md](./MILESTONE_1_TEST_REPORT.md) for verified evidence and remaining acceptance work.

CSV uploads use an all-or-nothing policy: if any row is invalid or duplicated, no rows from that file are imported. Correct the reported rows and upload again.

The app is intended for local development. Authentication is not implemented; do not expose it publicly or use real prospect data until the security and deployment requirements are addressed.
