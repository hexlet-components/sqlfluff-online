# Dependency Update Plan

## Branch: `update-dependencies`

## Strategy

Updates are split into **phases**. Each phase is one commit with `tox` run (black --check + pytest --cov=app at 100%). Phases go from safest to most risky, so if something breaks, it's immediately clear which dependency caused it.

---

## Phase 1 — Safe (all together)

| File | Package | Current | New |
|---|---|---|---|
| `requirements.txt` | `python-dotenv` | 1.0.1 | 1.2.2 |
| `requirements.txt` | `greenlet` | 3.1.1 | 3.5.1 |
| `requirements-dev.txt` | `coverage` | 7.6.1 | 7.14.1 |
| `requirements-dev.txt` | `requests` | 2.32.3 | 2.34.2 |
| `requirements-dev.txt` | `beautifulsoup4` | 4.12.3 | 4.14.3 |
| `requirements-dev.txt` | `tox` | 4.20.0 | 4.54.0 |

All patch/minor bumps, no API changes expected.

---

## Phase 2 — Flask 3.0.3 → 3.1.3

Minor update within 3.x series. No code changes required.

---

## Phase 3 — Flask-Limiter 3.8.0 → 4.1.1

Major version, but the app only uses the basic pattern:
- `Limiter(key_func=..., default_limits=[...])`
- `limiter.request_filter(...)`
- `limiter.init_app(app)`

These APIs are preserved in 4.x.

---

## Phase 4 — gevent 24.2.1 → 26.5.0

Major version, but gevent is known for API stability. Python 3.10 is still supported. No code changes in `wsgi.py`.

---

## Phase 5 — pytest 8.3.3 → 9.0.3 + pytest-cov 5.0.0 → 7.1.0

Updated together (pytest-cov 7.x requires pytest 7+). The test suite is straightforward — pytest 9 breaking changes don't apply here.

---

## Phase 6 — sqlfluff 3.2.0 → 4.2.1

**Main risk.** In 4.2.0:
- Python 3.9 dropped (app uses 3.10 — OK)
- New rules (AL10 and others)
- New `max_parse_nodes` setting
- Default `render_variant_limit` changed from 1 to 5

APIs `lint()`, `fix()`, `list_dialects()`, `list_rules()` are backward compatible.
Tests use concrete SQL queries; linting results may change slightly if new rules trigger.
**If it fails:** likely due to a new rule; test expectations may need adjustment.

---

## Phase 7 (optional) — Black 24.8.0 → 26.5.1

New "2026 stable style". `black . --check` will fail.
Steps:
1. Run `black .` to reformat all code
2. Verify with `git diff`
3. Commit formatting separately
4. Run `tox`

---

## Not in scope

- `flask-talisman` — no new version; needs replacement (unmaintained since 2021)
- Frontend CDN (Bootstrap 4→5 is breaking; jQuery, Ace, Popper can be updated separately)
- Python 3.10 → 3.12+ (requires Dockerfile, app.yaml, CI updates)

## Files to change

- `requirements.txt` — production dependencies
- `requirements-dev.txt` — dev dependencies

No source code changes expected (except possible test adjustments in Phase 6 and code formatting in Phase 7).

---

## How to verify

```bash
pip install -r requirements-dev.txt
tox
```

`tox` runs:
1. `black . --check --verbose`
2. `pytest -vv -rs --cov=app --cov-report=xml` (requires 100% coverage)

If a phase fails, roll back the last dependency change, investigate, and fix before proceeding.
