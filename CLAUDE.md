# CLAUDE.md

AskDB is an internal, read-only web app: non-technical staff ask plain-English questions and get a verified answer from a SQLite database.
Each response has an Answer, a Data table (with CSV download) and a "How I got this" explanation with the SQL behind a toggle.

## Status

Pre-implementation: only docs and sample data exist. The stack, commands and layout below are the plan from `docs/SPEC.md`.
Update this file as real tooling lands.

## Stack

- Backend: Python, FastAPI, stdlib `sqlite3`, `sqlglot`, Anthropic Python SDK. Managed with `uv`.
- Frontend: React (Vite).
- CLI: `askdb scaffold` and `askdb check` for schema config.

## Commands (planned; confirm against the real config once it exists)

- Install: `uv sync` and `npm install --prefix web`
- Run: `uv run uvicorn askdb.main:app --reload` and `npm run dev --prefix web`
- Test: `uv run pytest`, and one test with `uv run pytest tests/path.py::test_name`
- Eval (real model, not CI-gating): `uv run pytest tests/validation`
- Lint: `uv run ruff check .` and `npm run lint --prefix web`

## Layout

- `docs/SPEC.md`: v1 specification (source of truth). `docs/one-pager.md`: product summary. `docs/PLAN.md`: progress tracker.
- `data/`: sample DB `cloudmetrics.db`, its config `cloudmetrics.yaml`, golden questions `cloudmetrics_eval.yaml`.
- `src/askdb/`: backend (pipeline, SQL safety, answer verification, API, CLI), planned.
- `web/`: React frontend, planned.
- `tests/`: safety corpus, number validator, integration, eval, planned.

## Source of truth

- `docs/SPEC.md` decides behavior. If code and spec disagree, fix the code or change the spec deliberately and say so. Do not drift silently.
- `docs/PLAN.md` tracks progress. Update it in the same change that completes or adds a task.

## Guardrails

- SQL is read-only and layered (SPEC section 4): `mode=ro` connection with `query_only`, a `sqlglot` single-SELECT check, an authorizer limited to allow-listed columns, and a never-committed transaction. Never weaken or bypass a layer; never run client-supplied SQL.
- Every query has a row cap and a timeout (`ASKDB_DISPLAY_CAP`, `ASKDB_EXPORT_CAP`, `ASKDB_TIMEOUT_S`). Truncation must be shown to the user, never silent.
- Answer numbers must come from the returned rows; do arithmetic in SQL, not in answer text. Never return an unverified answer.
- Never commit secrets (API keys, passwords, session secrets). Config comes from environment variables.
- Never read `.env` files, and never echo secret values.
- The log DB (`askdb_log.db`) never stores result rows or answer text.
- Never hard-code the model ID; use `ASKDB_MODEL`.

## Conventions

- Every change comes with tests: new behavior gets new tests, and a bug fix gets a regression test. Run the tests before calling work done.
- Any change to SQL validation or the authorizer also adds cases to the adversarial safety corpus.

## Data gotchas

- `data/cloudmetrics.yaml` is a different format from the `schema.yaml` in SPEC section 3. Reconcile it when writing the loader.
- Eval cases pass on matching result sets, not SQL text, and use explicit date ranges because the data ends in 2025-11.
- MRR = `SUM(subscriptions.mrr)` where `status='active'` (already monthly). Revenue = paid invoices only. Churn is per subscription.
