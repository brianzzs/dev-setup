---
name: python-conventions
description: >
  Python conventions. Use for any Python work: reading project configuration,
  preserving the pinned version and package manager, following the configured
  formatter, linter, and type checker, async discipline, and test selection.
---

# Python

- Read `pyproject.toml`, version files, lockfiles, environment configuration, and nearby code first. Preserve the pinned Python version and the repository's package manager (`uv`, Poetry, pip, or another configured tool).
- Follow the configured formatter, linter, and type checker. Prefer clear type hints at public and boundary APIs without annotation noise the project cannot enforce.
- Prefer `pathlib`, context managers, explicit resource ownership, timezone-aware datetimes, and standard-library solutions when they improve clarity.
- Never use mutable default arguments. Catch specific exceptions, never bare or broad `except`, and preserve traceback and context when translating.
- Keep async code end to end when using an async framework; do not block the event loop with synchronous network, file, or subprocess work.
- Use the repository's test conventions. Run the narrowest focused `pytest` or configured runner target first, then broader tests. Run Ruff, Black, mypy, or Pyright only when the project configures them.
