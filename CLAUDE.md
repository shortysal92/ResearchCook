# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
make install        # uv sync --all-extras
make check          # ruff format --check + ruff lint (CI-safe, no writes)
make fix            # ruff format + ruff check --fix (auto-fixes everything)
make test           # pytest
make clean          # remove __pycache__, .pytest_cache, .ruff_cache
```

Run a single test file:
```bash
uv run pytest tool_use/tests/test_memory_tool.py
```

Validate notebook structure before committing (also runs as a pre-commit hook):
```bash
uv run python scripts/validate_notebooks.py path/to/notebook.ipynb
```

Execute a notebook to test it (requires `ANTHROPIC_API_KEY`):
```bash
uv run jupyter nbconvert --to notebook --execute path/to/notebook.ipynb \
  --ExecutePreprocessor.kernel_name=python3 --output executed.ipynb
```

## Architecture

### This is an educational notebook repository, not a Python application

The primary artifacts are ~59 Jupyter notebooks. `anthropic_cookbook/` (Python package) and `pyproject.toml` exist solely as build scaffolding — the package itself is a dummy with only `__init__.py`. Python utility files (`tool_use/memory_tool.py`, `skills/file_utils.py`, etc.) exist to support specific notebooks, not as a standalone library.

### Claude Code slash commands run in both local sessions and CI

`.claude/commands/` defines three slash commands (`/notebook-review`, `/model-check`, `/link-review`). These are invoked identically in local Claude Code and in GitHub Actions via `anthropics/claude-code-action@v1`. Running them locally before pushing exercises the same logic CI will run.

### Pre-commit hooks enforce notebook cleanliness

`.pre-commit-config.yaml` runs `ruff` (format + lint) and `scripts/validate_notebooks.py` on every commit. The validator rejects notebooks with empty cells or error outputs — clear outputs and delete empty cells before committing.

### Notebook outputs are intentionally kept in the repo

Unlike most projects, cell outputs are committed. They serve as documentation showing expected results. Do not strip them.

### `claude_agent_sdk/` is a separate Python project

It has its own `pyproject.toml` and must be installed independently:
```bash
cd claude_agent_sdk && uv sync
uv run python -m ipykernel install --user --name="cc-sdk-tutorial"
```
It also requires `node` and the Claude Code CLI (`npm install -g @anthropic-ai/claude-code`).

### `skills/` has its own `CLAUDE.md` with critical gotchas

Skills use the `client.beta.*` namespace and require specific beta headers per-request (not on the client constructor). File content is retrieved with `.read()`, not `.content`. File size uses `.size_bytes`. Document generation takes 1–2 minutes. See `skills/CLAUDE.md` for full details.

## Code Conventions

- **Linter/formatter**: `ruff` (line length 100, Python 3.11+, double quotes)
- Notebooks suppress `N806`, `E402`, `F811`, `N803` — these are expected in notebook-style code
- Every notebook should define a `MODEL` constant at the top and load credentials with `dotenv.load_dotenv()`
- Notebook files use `kebab-case.ipynb` or `00_numbered_name.ipynb`; Python files use `snake_case.py`

## Current Claude Models

- `claude-opus-4-6` — most capable
- `claude-sonnet-4-6` — balanced (default for most examples)
- `claude-haiku-4-5-20251001` — fast/lightweight

The `/model-check` command and `claude-model-check.yml` CI workflow flag deprecated model IDs.
