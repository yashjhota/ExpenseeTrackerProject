# Expense Tracker

Personal expense tracker web app. Single-user, offline-first, Python backend, SQLite storage (cloud-synced later).

## Scope (v1)

- **Users:** Single user only (you). No auth, no multi-user, no sharing.
- **Access:** Web app, accessed locally (localhost). Offline-first — runs against a local SQLite file; no network required to add/view expenses.
- **Entry:** Manual entry only (no bank/card import in v1).
- **Storage:** SQLite file on disk (first version). Cloud sync is a planned v2 migration, not part of v1.
- **Data stays local:** Nothing is sent to a third party.

## Core Features (v1)

1. **Expenses CRUD** — add, edit, delete, list expenses.
2. **Categories** — assign each expense to a category (name + optional color). Category CRUD too.
3. **Monthly budgets** — set a budget per category per month; track spend vs. budget.
4. **Multi-currency** — each expense carries a currency code (default your local currency). No FX conversion in v1.
5. **Recurring expenses** — mark an expense as repeating (daily/weekly/monthly). Due items are generated on app startup (and can be regenerated anytime with the `generate-recurring` CLI command).
6. **Reports & export** — monthly spending view, CSV export, PDF export, printable report page.
7. **Receipts** — optional file attachment path/note per expense (store path only; no file handling in v1 unless noted).

Out of scope for v1: user auth, bank import, cloud sync, mobile app, FX conversion, receipt image upload/OCR.

## Tech Stack (decided — fixed for v1)

| Layer | Choice | Notes |
|-------|--------|-------|
| Language | Python 3.11+ | |
| Web framework | FastAPI | ASGI, server-rendered Jinja2 templates, good offline fit |
| Database | SQLite | Single local file, `expense_tracker/instance/expensee.db` |
| Frontend | Server-rendered Jinja2 templates + vanilla JS | No build step, no npm, no CDN |
| Data validation | Pydantic (built-in) | FastAPI-native |
| PDF reports | ReportLab | Installs with pip alone; no system libs needed |
| Testing | pytest | |
| Linting | ruff or black | |

If you want a different framework or PDF lib, say so before coding.

## Project Structure

```
expense_tracker/
  app.py              # FastAPI app + CLI entry (init-db, seed, export-csv, generate-recurring)
  models.py           # SQLAlchemy or raw-sqlite models
  database.py         # DB connection / schema init
  routes/             # route modules using APIRouter (expenses, categories, budgets, reports)
  templates/          # Jinja2 HTML templates
  static/             # CSS, JS, images (vendored locally — no CDN)
  instance/           # SQLite DB file (gitignored)
tests/
  test_*.py
requirements.txt
CLAUDE.md
.gitignore
```

Use `APIRouter` per domain area (FastAPI's equivalent of Flask blueprints). Keep business logic in `models.py` or a `services/` module — not in route handlers.

## Data Model (v1)

- `categories` — id, name, color, created_at
- `expenses` — id, category_id, amount, currency, date, description, notes, recurring (bool), recurring_interval, created_at, updated_at
- `budgets` — id, category_id, month (YYYY-MM), amount, currency, created_at

Schema lives in `database.py` (or an Alembic migration if you prefer). Keep migrations minimal for v1.

## Development Commands

```bash
# setup
pip install -r requirements.txt

# init DB (create tables)
python -m expense_tracker init-db

# run dev server (offline, localhost)
uvicorn expense_tracker.app:app --reload --host 127.0.0.1 --port 5000

# run tests
pytest

# lint
ruff check .    # or black --check
```

CLI commands in `app.py`: `init-db`, `seed`, `export-csv`, `generate-recurring`.

## Recurring Expenses (decision)

No scheduler and no network, so recurring items are **generated on app startup** (startup hook in `app.py`). They can also be regenerated anytime with the `generate-recurring` CLI command. This keeps behavior deterministic and offline.

## Reporting

- Monthly aggregate: total per category, spend vs. budget, currency totals.
- CSV export: all expenses with filters (date range, category, currency).
- PDF export: printable monthly report (ReportLab).
- Printable view: CSS `@media print` stylesheet hiding nav/controls.

## Rules & Conventions

1. **Offline first:** Every feature must work with no network. No CDN assets — vendor any CSS/JS in `static/`.
2. **Single user:** No `current_user`, sessions, or login routes. If you add auth later, it's a v2 branch.
3. **SQLite is the source of truth:** No separate in-memory cache that can drift. Read/write through `database.py`.
4. **Dates:** ISO 8601 (`YYYY-MM-DD`) in DB and API; display localized in templates.
5. **Money:** Store as integer cents (or `Numeric`) to avoid float errors. Currency is a separate column.
6. **No external services:** No Plaid, no email, no cloud. v1 is a local-only desktop-grade web app.
7. **Tests first for reports:** CSV/PDF export logic must have unit tests before it's wired to routes.

## Future (v2+)

- Cloud sync (PostgreSQL hosted) — requires auth + user model
- Bank/card CSV import
- Receipt image uploads
- FX conversion
- Mobile-responsive / PWA

## Workflow (spec-driven)
- No code without an approved spec and plan in specs/<NN-feature>/
- Flow: /spec → /plan → /tasks → /implement → /ship. Stop for my approval after each.
- One feature = one git branch (feat/<name>) = one PR
- Implement ONE task at a time; run `pytest` and `ruff check .` before marking it done
- Commit after each task using Conventional Commits (feat:, fix:, test:, docs:, chore:)
- Never push to main. Never force-push. Never commit instance/ or .env
- If code and spec disagree, stop and ask

## How to use this CLAUDE.md

This is the single source of truth for the project's scope and stack. Before writing code, check here. If you want to change the framework, add auth, or move to cloud storage, edit this file first and we'll agree on the change before implementing.
