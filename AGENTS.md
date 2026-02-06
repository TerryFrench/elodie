# AGENTS.md

## Dev environment tips
- This project is a Python CLI. Main entrypoint: `./elodie.py`. Core logic lives in `elodie/`.
- Default working scope: repository root and `elodie/`. Ignore `app/` and `node_modules/` unless explicitly requested.
- Recommended setup:
  - `python3 -m venv .venv && source .venv/bin/activate`
  - `pip install -r requirements.txt`
  - `pip install -r elodie/tests/requirements.txt`
- ExifTool is a required external dependency for runtime and tests. Verify with `exiftool -ver`.
- Optional plugin dependencies:
  - `pip install -r elodie/plugins/googlephotos/requirements.txt`
- Elodie stores runtime state under `~/.elodie/` by default (`hash.json`, `location.json`, config). You can override with `ELODIE_APPLICATION_DIRECTORY`.

## Testing instructions
- Run the main test suite:
  - `pytest elodie/tests -v`
- CI-style run with coverage:
  - `pytest elodie/tests -v --cov=elodie --cov-report=xml`
  - Coverage flags require `pytest-cov`:
  - `python3 -m pip install --user --break-system-packages pytest-cov`
- Lint package code:
  - `flake8 elodie --exclude=tests,external`
- Tests use `elodie/tests/conftest.py` to create an isolated temporary app directory for each test.
- Geolocation tests require a valid MapQuest key in environment:
  - `export MAPQUEST_KEY=write_the_real_key`
- Live MapQuest integration tests cannot be validated from Codex sandbox due network restrictions in this environment; run geolocation-related pytest commands in your own shell.

## PR instructions
- Keep patches focused and limited to requested scope.
- For behavior changes, update or add tests under `elodie/tests/`.
- For commands that modify files (`import`, `update`, `generate-db`), validate with a `--dry-run` command when possible.
- Do not commit secrets or real API keys to tracked files (especially `config.ini`).
- If adding new dependencies or env vars, document them in `Readme.md`.

## Project notes
- CLI commands in `elodie.py`: `import`, `update`, `generate-db`, `verify`, `batch`.
- External dependency checks are in `elodie/dependencies.py` (notably `exiftool`).
- Configuration loading is handled via `elodie/config.py` and expects `config.ini` in the application directory.

## Current tracked problems
- `#1 Unicode filenames can crash import/update flows` (status: core fix implemented; targeted tests passing)
- `#2 Media with pre-Unix-epoch dates are not handled robustly` (status: core fix implemented; targeted tests passing)

### Notes
- Unicode hardening was added at the ExifTool/media boundary to avoid crashes when metadata decoding fails.
- End-to-end unicode import tests in `elodie/tests/elodie_test.py` are environment-sensitive for geolocation (`MAPQUEST_KEY`) and can fail with `Unknown Location` while still validating that unicode paths do not crash.
- Pre-epoch date handling now avoids hard failures from `time.mktime`/`os.utime` on platforms that do not support negative epoch values.
- Test tidying completed (separate follow-up commit):
  - Rewrote `test_various_types` in `elodie/tests/media/photo_test.py` using `pytest.mark.parametrize`.
  - Removed redundant ExifTool module-level setup in tests to avoid duplicate-start warnings from the singleton process.

## Storage safety roadmap
- Near-term option (now): keep JSON files (`hash.json`, `location.json`) but improve resilience with atomic writes and graceful interrupt handling.
- Near-term option (opt-in): add periodic hash DB flush batching for long imports to trade durability granularity for better throughput.
- Mid-term option: migrate hash/location state to SQLite with transactions and indexed lookups to improve crash-safety, incremental updates, and scale.

## Current plan
- Commit 1: add OS-independent graceful `SIGINT`/`Ctrl+C` handling in CLI loops (`import`, `update`, `generate-db`, `verify`) so runs stop cleanly and exit with interrupt code.
- Commit 2: add opt-in import hash DB batching via CLI flag and switch JSON DB writes to atomic replace semantics in `localstorage.py`.
