````markdown
# COPILOT — Developer Companion

Purpose
-------

This document provides quick onboarding, development, testing, and PR guidance for contributors working on this fork of Elodie. It's a concise complement to the project's main README and an adaptation of the project's internal `AGENTS.md` template targeted for interactive work with a coding assistant (Copilot).

Scope & focus
-------------

- Primary focus: files in the repository root and the `elodie/` Python package.
- Ignore: `app/` and `node_modules/` unless specifically requested.

Quick links
-----------

- Main CLI entry: `elodie.py`
- Core package: `elodie/`
- Tests: `elodie/tests/`

Dev environment (recommended)
-----------------------------

1. Create and activate a virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

2. Install runtime + test requirements:

```bash
pip install -r requirements.txt
pip install -r elodie/tests/requirements.txt
```

3. Verify external dependency ExifTool is available:

```bash
exiftool -ver
```

Testing
-------

- Run the full test suite:

```bash
pytest elodie/tests -v
```

- CI-style with coverage:

```bash
pytest elodie/tests -v --cov=elodie --cov-report=xml
```

Linting
-------

- Use `flake8` for basic style checks on the package:

```bash
flake8 elodie --exclude=tests,external
```

PR & contribution guidelines
----------------------------

- Keep patches focused and minimal; change only what's requested.
- For behavior changes, add or update tests under `elodie/tests/`.
- Avoid committing secrets or API keys (e.g., MapQuest or Google Photos creds).
- For commands that modify files, prefer implementing a `--dry-run` mode and test it.

Project notes (important)
-------------------------

- External dependencies are checked in `elodie/dependencies.py` (ExifTool is required).
- Configuration defaults and logic live in `elodie/config.py` and `elodie/config.ini`.
- Runtime state is stored under `~/.elodie/` by default (can be overridden with `ELODIE_APPLICATION_DIRECTORY`).

Where to look first
-------------------

- Entrypoint: see `elodie.py` for CLI wiring.
- Core functionality: the `elodie/` package (especially `media/`, `filesystem.py`, and `geolocation.py`).
- Tests and fixtures: `elodie/tests/` and `elodie/tests/conftest.py` for example usage and expected behaviors.

Notes for the Copilot assistant
-----------------------------

- Focus editing and analysis on the root and `elodie/` files by default.
- When asked to change behavior or add features, include or update tests.
- Keep changes minimal and revertible; aim to fix root causes, not symptoms.

Next steps
----------

- I created this initial `COPILOT.md` to capture the essentials from `AGENTS.md` and the repository's attachments. If you'd like, I can now:
  - Expand any section with more details (examples, more commands).
  - Add contributor checklist items or a small developer quickstart script.
  - Proceed to the code modifications you mentioned.

**Tracked Issues**

- **Issue #1 (Unicode filenames):** Some photos contain Unicode characters in their filenames which can cause crashes during processing (reading, metadata handling, or database writes). Status: **open**.
- **Issue #2 (Pre-Unix-epoch dates):** Some photos have `DateTaken` values prior to the Unix epoch (1970-01-01) which causes `time.mktime()` or `os.utime()` to fail. Status: **open**.

**Plan — Problem #1: Unicode filenames (first pass)**

- **Goal:** Avoid crashes when encountering filenames with non-ASCII or otherwise unusual Unicode characters. Ensure consistent handling across file discovery, metadata parsing, database storage, and JSON serialization.

- **Steps:**
  1. Identify all code paths that accept or store file paths and names (`elodie/filesystem.py`, `elodie/localstorage.py`, `external/pyexiftool.py`, media classes). Add a short unit test that imports a file with a Unicode filename to reproduce the crash.
  2. Normalize incoming path objects early: call `os.fspath()` / `os.fsdecode()` where necessary to guarantee `str` (text) values. When interacting with APIs that might return bytes, decode using the filesystem encoding with `errors="surrogateescape"`.
  3. When writing database JSON files, call `json.dump(..., ensure_ascii=False)` to preserve Unicode and avoid encoding surprises. When reading, accept escaped sequences and handle them via standard `json.load`.
  4. Wrap encoding-sensitive calls (printing, logging, regex, and `re.sub`) in try/except to convert problematic values using a safe fallback (e.g., `repr()` or `errors='replace'`) so the program continues without crashing.
  5. Add tests under `elodie/tests/` covering: discovery via `get_all_files()`, `process_file()` round-trip, and `Db.update_hash_db()` writing with Unicode keys/values.

- **Acceptance criteria:**
  - Importing a file whose filename contains non-ASCII characters does not raise UnicodeEncodeError or similar exceptions during discovery, hashing, DB update, or move/copy operations.
  - The `~/.elodie/hash.json` file is valid JSON and includes the file path (Unicode intact).
  - New unit tests covering the unicode-filename case pass.

**Next:**

- I'll implement small, focused changes for problem #1: add `os.fsdecode()` calls where file paths are consumed and update `Db.update_hash_db()` / `Db.update_location_db()` to use `json.dump(..., ensure_ascii=False)`. I will then add unit tests to reproduce and verify the fix.

--
Generated and seeded from the repository's `AGENTS.md` template.

````