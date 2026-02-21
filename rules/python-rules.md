---
name: Python Rules
dcc_uri: dev/rules/python-rules
description: Python development rules (Python 3.11+). Extends core rules.
version: '1.0.0'
dcc_definition_type: rule
dcc_tags:
  - dev
  - backend
  - python
---

# Python Rules (Python 3.11+)

These rules apply to all Python projects and extend the core rules.

---

## Types & Contracts

- Annotate every function and method with parameter types and return type. No exceptions, including `__init__` (return type is `None`) and private methods.
- Use `X | None` (union syntax) instead of `Optional[X]` for nullable types — it is cleaner and available from Python 3.10+.
- Use `typing.TypeAlias` for complex type aliases; give them descriptive names rather than inlining long types repeatedly.
- Never use `Any` unless integrating with a truly untyped external boundary and even then, isolate it to the smallest possible scope.
- Run `mypy` or `pyright` in strict mode. Type annotations are not documentation — they are a contract the type checker must verify.

---

## Error Handling

- Never use bare `except:` — always catch a specific exception type or at minimum `except Exception`.
- Never silently swallow exceptions with an empty `except` block. Log and re-raise, or handle explicitly.
- Raise the most specific built-in exception that fits (`ValueError`, `TypeError`, `KeyError`, etc.) before reaching for a custom exception. Create custom exceptions only when the caller needs to distinguish them programmatically.
- Use `contextlib.suppress` only for truly ignorable exceptions, never as a shortcut for lazy error handling.
- Do not use exceptions for flow control. Exceptions are for unexpected conditions; use `if` for expected branching.

---

## Patterns & Idioms

- Use dataclasses (`@dataclass`) or `pydantic.BaseModel` for structured data. Do not use plain dicts to pass around multi-field data — they have no schema, no type safety, and no discoverability.
- Use `pathlib.Path` for all file system operations. Never use `os.path` string manipulation in new code.
- Use context managers (`with`) for any resource that needs cleanup: files, connections, locks. Never rely on garbage collection to close resources.
- Prefer `Enum` over string or integer constants for closed sets of values.
- Use `__slots__` on classes that will be instantiated many times and do not need a dynamic `__dict__`.

---

## Performance & Safety

- Never mutate a default argument. `def fn(items=[])` is a well-known Python trap — use `None` as the default and assign inside the function.
- Never use `eval()` or `exec()` on external or user-supplied input.
- Do not use `global` or `nonlocal` state unless there is a clear, documented reason. Prefer passing values explicitly.
- For CPU-bound work, use `multiprocessing` or `concurrent.futures.ProcessPoolExecutor` — threads do not parallelize CPU work due to the GIL. For I/O-bound work, use `asyncio` or `ThreadPoolExecutor`.
- Cache expensive pure function results with `functools.lru_cache` or `functools.cache`. Do not re-compute what can be memoised.

---

## Documentation

- Write Google-style docstrings for all public functions, methods, and classes. Include: a one-line summary, `Args:`, `Returns:`, and `Raises:` sections where applicable.
- Docstrings document the contract (what, why, preconditions, exceptions) — not the implementation. Do not restate what the code already clearly shows.

---

## Project & Dependencies

- Every project must have a `pyproject.toml` as the single source of build and tool configuration. Do not use `setup.py`, `setup.cfg`, or scattered `*.cfg` files for new projects.
- Pin all dependencies with exact versions in a lockfile (`pip-compile`, `poetry.lock`, `uv.lock`). Never deploy with unpinned dependencies.
- Use a virtual environment per project. Never install project dependencies into the system Python.
- Structure imports in three groups separated by blank lines: standard library → third-party → local. Use `isort` or `ruff` to enforce this automatically.
