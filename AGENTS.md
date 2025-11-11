<!-- OPENSPEC:START -->
# OpenSpec Instructions

These instructions are for AI assistants working in this project.

Always open `@/openspec/AGENTS.md` when the request:
- Mentions planning or proposals (words like proposal, spec, change, plan)
- Introduces new capabilities, breaking changes, architecture shifts, or big performance/security work
- Sounds ambiguous and you need the authoritative spec before coding

Use `@/openspec/AGENTS.md` to learn:
- How to create and apply change proposals
- Spec format and conventions
- Project structure and guidelines

Keep this managed block so 'openspec update' can refresh the instructions.

<!-- OPENSPEC:END -->

# Repository Guidelines

## Project Structure & Module Organization
- `elysia/` — Python package source. Key areas: `api/` (FastAPI app, CLI, routes), `tools/` (retrieval, text, postprocessing, visualisation), `tree/` (decision tree core), `preprocessing/`, `util/`.
- `tests/` — Pytest suite split into `no_reqs/` (default, no external deps) and `requires_env/` (needs API keys; may incur LLM costs). See `tests/requires_env/README.md`.
- `docs/` — MkDocs documentation sources. `mkdocs.yml` config at repo root.
- `.github/` — CI workflows and issue/PR templates.
- `img/` — Documentation assets.

## Build, Test, and Development Commands
- Setup (Python 3.10–3.12): `python -m venv .venv && source .venv/bin/activate`
- Install dev deps (editable): `pip install -e ".[dev]"`
- Run API locally: `elysia start --host 0.0.0.0 --port 8000 --reload`
- Run tests (default): `pytest tests/no_reqs -q`
- Full tests with coverage: `pytest --cov=elysia --cov-report=term-missing`
- Docs preview/build: `mkdocs serve` / `mkdocs build`

## Coding Style & Naming Conventions
- Format with Black (PEP 8, 4-space indent): `black elysia tests`
- Use type hints and concise docstrings. Keep modules lowercase with underscores; classes `PascalCase`; functions/vars `snake_case`.
- Prefer pure functions and small units in `util/` and `tools/`; keep API concerns inside `elysia/api/`.

## Testing Guidelines
- Frameworks: `pytest`, `pytest-asyncio`, optional `pytest-cov`.
- Test naming: files `test_*.py`; functions `test_*`. Co-locate fixtures in `conftest.py`.
- Default CI target is `tests/no_reqs/`. `requires_env/` needs `.env` with `WCD_URL`, `WCD_API_KEY`, `OPENROUTER_API_KEY`, `OPENAI_API_KEY` and will call LLMs (costs may apply).

## Commit & Pull Request Guidelines
- Branches: features → `main`; fixes/chores → current `release/vY.Z.x` (see CONTRIBUTING.md). Prefix branches like `feature/...`, `bugfix/...`.
- Commits: imperative present (“Add X”), scoped and focused. Reference issues (`Fixes #123`).
- PRs: clear description, linked issues, before/after notes for API/behavior changes, tests updated, docs updated when needed, CI green.

## Security & Configuration
- Use a local `.env`; never commit secrets. Keys above are only for `requires_env/` tests and local experimentation.
