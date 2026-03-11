# ResearchCook (Anthropic Cookbook) - Claude Code Guide

## Project Overview

ResearchCook is the **Anthropic Cookbook** — a comprehensive educational repository of Jupyter notebooks showcasing Claude's capabilities through practical, production-ready examples. It covers agents, RAG, multimodal, tool use, extended thinking, skills (document generation), and third-party integrations.

**Key facts:**
- ~59 Jupyter notebooks organized by capability
- ~62 Python utility files
- 7 GitHub Actions CI/CD workflows
- 3 custom Claude Code slash commands + 1 custom skill
- MIT License, Python 3.11+, `uv` package manager

---

## Quick Start

```bash
# Install dependencies (uv is the required package manager)
make install          # runs: uv sync --all-extras

# Install pre-commit hooks
uv run pre-commit install

# Copy and configure environment
cp .env.example .env
# Add ANTHROPIC_API_KEY to .env

# Launch Jupyter
jupyter notebook
```

---

## Repository Structure

```
ResearchCook/
├── .claude/
│   ├── commands/             # Slash commands: /link-review, /model-check, /notebook-review
│   └── skills/cookbook-audit/  # Custom cookbook audit skill + style guide
├── .github/workflows/        # 7 CI/CD workflows
├── anthropic_cookbook/       # Dummy Python package (build artifact only)
├── capabilities/             # Core capability guides with evaluations
│   ├── classification/
│   ├── contextual-embeddings/
│   ├── retrieval_augmented_generation/
│   ├── summarization/
│   └── text_to_sql/
├── claude_agent_sdk/         # Claude Agent SDK tutorial series (3 notebooks)
│   ├── 00_The_one_liner_research_agent.ipynb
│   ├── 01_The_chief_of_staff_agent.ipynb
│   ├── 02_The_observability_agent.ipynb
│   ├── research_agent/       # Implementation backing notebook 00
│   ├── chief_of_staff_agent/ # Implementation backing notebook 01
│   ├── observability_agent/  # Implementation backing notebook 02
│   └── utils/
├── coding/                   # Frontend design guidance notebook
├── extended_thinking/        # Extended thinking examples (2 notebooks)
├── finetuning/               # Fine-tuning examples
├── misc/                     # 13 diverse standalone recipes
├── multimodal/               # Vision and multimodal examples (5 notebooks)
├── observability/            # Cost and usage API examples
├── patterns/agents/          # Prompt patterns for agentic systems
├── scripts/
│   ├── validate_notebooks.py       # Notebook structure validator
│   ├── validate_all_notebooks.py   # Comprehensive validator
│   └── detect-secrets/            # Secret detection baseline
├── skills/                   # Document generation cookbook (Excel, PPT, PDF)
│   ├── notebooks/            # 3 progressive notebooks
│   ├── custom_skills/        # Custom skill development examples
│   ├── sample_data/          # Financial datasets (CSV, JSON)
│   ├── file_utils.py         # Files API helper functions
│   ├── skill_utils.py        # Skill utilities
│   └── CLAUDE.md             # Detailed skills-specific guide
├── third_party/              # 8 third-party integrations
│   ├── Deepgram/ ElevenLabs/ LlamaIndex/ MongoDB/
│   ├── Pinecone/ VoyageAI/ Wikipedia/ WolframAlpha/
├── tool_use/                 # Tool integration examples (6 notebooks)
│   ├── memory_tool.py        # Production-ready memory handler
│   ├── memory_demo/
│   └── tests/
├── tool_evaluation/          # Tool evaluation notebook
├── .env.example              # Environment variables template
├── .pre-commit-config.yaml   # Pre-commit hook config
├── lychee.toml               # Link validation config
├── Makefile                  # Dev targets (format, lint, test, check, fix)
├── pyproject.toml            # Project metadata and build config
├── requirements.txt          # Runtime dependencies
├── requirements-dev.txt      # Development dependencies
├── uv.lock                   # Lock file (do not edit manually)
└── uv.toml                   # uv configuration
```

---

## Development Workflows

### Makefile Targets

```bash
make install    # uv sync --all-extras
make format     # ruff format (applies formatting)
make lint       # ruff check (reports issues)
make check      # format-check + lint (no changes, CI-safe)
make fix        # ruff format + ruff check --fix (auto-fixes all)
make test       # pytest
make clean      # remove __pycache__, .ruff_cache, etc.
```

### Running Quality Checks

```bash
# Validate notebook structure (required before PR)
uv run python scripts/validate_notebooks.py

# Run all pre-commit hooks on staged files
uv run pre-commit run

# Run all pre-commit hooks on all files
uv run pre-commit run --all-files

# Lint + format a specific directory
uv run ruff check skills/ --fix
uv run ruff format skills/
```

### Running Tests

```bash
make test                    # runs pytest
uv run pytest                # equivalent
uv run pytest tool_use/tests/  # run specific tests
```

### Executing a Notebook (optional — requires API key)

```bash
uv run jupyter nbconvert \
  --to notebook \
  --execute path/to/notebook.ipynb \
  --ExecutePreprocessor.kernel_name=python3 \
  --output test_output.ipynb
```

---

## Claude Code Slash Commands

These commands work identically in local Claude Code sessions and in GitHub Actions CI:

| Command | Purpose |
|---|---|
| `/notebook-review <path>` | Comprehensive notebook quality check |
| `/model-check` | Verify all Claude model references are current |
| `/link-review <path>` | Validate links in markdown/notebooks for quality and security |

Command definitions live in `.claude/commands/`. Run them before pushing to catch the same issues CI will flag.

There is also a **cookbook-audit** custom skill in `.claude/skills/cookbook-audit/` with a full style guide rubric (`SKILL.md` and `style_guide.md`).

---

## Code Conventions

### Python Style (enforced by ruff)

- **Formatter**: ruff (line length: 100)
- **Target Python**: 3.11+
- **Quotes**: double quotes
- **Indent**: 4 spaces
- **Ignored rules**:
  - `N806` globally (API response variable names may be non-snake-case)
  - `E402`, `F811`, `N803`, `N806` per-file for notebooks (different execution model)

### Naming Conventions

- Variables/functions: `snake_case`
- Classes: `PascalCase`
- Constants: `UPPER_SNAKE_CASE`
- Python files: `snake_case.py`
- Notebook files: `kebab-case.ipynb` or numbered prefix `00_name.ipynb`

### API Key Handling

**Never hardcode API keys.** Always use environment variables:

```python
import os
from dotenv import load_dotenv

load_dotenv()
api_key = os.environ.get("ANTHROPIC_API_KEY")
```

### Current Claude Models

Always use current, valid model IDs. Claude CI will automatically flag outdated models. Valid models as of this writing:

- `claude-opus-4-6` — Most capable
- `claude-sonnet-4-6` — Balanced performance (default for most examples)
- `claude-haiku-4-5-20251001` — Fast and lightweight

Check the latest at: https://docs.claude.com/en/docs/about-claude/models/overview

### Notebook Conventions

1. **One concept per notebook** — keep scope focused
2. **Define `MODEL` constant at top** of the notebook
3. **Suppress pip install output** with `%%capture`
4. **Include expected outputs** as markdown cells explaining what to expect
5. **Load credentials with dotenv** — use `load_dotenv()` not hardcoded values
6. **Run top-to-bottom without errors** before committing
7. **Keep API calls minimal** in examples (use `max_tokens` limits)
8. **Notebook outputs are kept** in the repo — they demonstrate expected results for users

---

## Git Workflow

### Branch Naming

```
<your-name>/<feature-description>
# Example: alice/add-rag-example
```

### Conventional Commits

```
<type>(<scope>): <subject>

Types:
  feat      New feature or notebook
  fix       Bug fix
  docs      Documentation only
  style     Formatting, no logic change
  refactor  Code restructuring
  test      Adding or fixing tests
  chore     Maintenance tasks
  ci        CI/CD changes

Examples:
  feat(skills): add text-to-sql notebook
  fix(api): use environment variable for API key
  docs(readme): update installation instructions
  chore(deps): bump anthropic to 0.71.0
```

### Before Pushing a PR

- [ ] Restart kernel and run all cells top-to-bottom without errors
- [ ] Run `uv run python scripts/validate_notebooks.py`
- [ ] Run `make check` (format and lint)
- [ ] Verify no hardcoded API keys or secrets
- [ ] Check model IDs are current with `/model-check`
- [ ] Test links with `/link-review` if you added URLs

---

## Dependencies

### Runtime (`requirements.txt` / `pyproject.toml`)

| Package | Version | Purpose |
|---|---|---|
| anthropic | >=0.71.0 | Core Claude SDK (required for Skills support) |
| jupyter | >=1.1.1 | Notebook environment |
| ipykernel | >=7.1.0 | Jupyter kernel |
| notebook | >=7.4.7 | Notebook server |
| numpy | >=2.3.4 | Numerical computing |
| pandas | >=2.3.3 | Data manipulation |
| voyageai | >=0.3.5 | Embedding models |

### Development (`requirements-dev.txt`)

| Package | Version | Purpose |
|---|---|---|
| papermill | >=2.6.0 | Programmatic notebook execution |
| ruff | >=0.14.2 | Linting and formatting |
| pytest | >=8.3.3 | Test framework |
| nbval | >=0.11.0 | Notebook output validation |
| pre-commit | >=3.8.0 | Git pre-commit hooks |
| nbconvert | >=7.16.0 | Notebook conversion/execution |

### Claude Agent SDK (inside `claude_agent_sdk/`)

- `claude-agent-sdk>=0.1.6`
- `mcp-server-git>=2025.1.14`
- `python-dotenv>=1.1.1`

---

## CI/CD Workflows

All workflows are in `.github/workflows/`:

| Workflow | Trigger | Purpose |
|---|---|---|
| `lint-format.yml` | PR / push | ruff lint and format check |
| `notebook-quality.yml` | PR | Notebook structure validation |
| `links.yml` | PR / schedule | Link validation with lychee |
| `claude-notebook-review.yml` | PR | AI-powered notebook code review |
| `claude-model-check.yml` | PR | Validate model IDs are current |
| `claude-link-review.yml` | PR | AI-powered link quality review |
| `notebook-diff-comment.yml` | PR | Posts notebook diffs as PR comments |

CI uses Python 3.11, caches dependencies, and only runs checks on changed files.

---

## Environment Variables

```bash
# Required
ANTHROPIC_API_KEY=sk-ant-api03-...

# Optional — skills subsection
ANTHROPIC_BASE_URL=https://api.anthropic.com  # for proxies

# Optional — Claude Agent SDK notebooks
GITHUB_TOKEN=ghp_...                           # GitHub integration

# Optional — for test harnesses
CLAUDE_MODEL=claude-haiku-4-5-20251001
TEST_MODE=true
MAX_TOKENS=10
DEBUG=false
```

Copy `.env.example` to `.env` and fill in values. Never commit `.env`.

---

## Subsection-Specific Notes

### Skills (`skills/`)

See `skills/CLAUDE.md` for the full guide. Key points:

- Requires `anthropic>=0.71.0`
- All Skills functionality uses the `client.beta.*` namespace
- Required per-request betas: `["code-execution-2025-08-25", "files-api-2025-04-14", "skills-2025-10-02"]`
- File IDs are extracted from `bash_code_execution_tool_result.content`; use `file_utils.extract_file_ids(response)`
- Document generation takes 1-2 minutes — warn users in notebook markdown cells
- Use `client.beta.files.download(file_id).read()` (not `.content`) for file content
- Use `metadata.size_bytes` (not `.size`) for file metadata

### Claude Agent SDK (`claude_agent_sdk/`)

- Has its own `pyproject.toml` — install separately with `uv sync` inside that directory
- Three progressive notebooks: research agent → chief of staff → observability
- Requires `GITHUB_TOKEN` for observability agent
- Uses MCP server for git integration

### Capabilities (`capabilities/`)

Each capability has:
- A main guide notebook
- An evaluation subdirectory with test files
- A README explaining the use case

### Third-Party (`third_party/`)

Each integration is self-contained with its own README and requirements. Check the subdirectory README before working on integrations — they may have additional API key requirements.

---

## Testing

```bash
# Unit tests (Python files only)
uv run pytest

# Test specific module
uv run pytest tool_use/tests/test_memory_tool.py

# Validate notebook structure (no API calls needed)
uv run python scripts/validate_notebooks.py

# Full pre-commit suite
uv run pre-commit run --all-files
```

**Note**: Full notebook execution tests require a valid `ANTHROPIC_API_KEY` and consume API credits. They are typically run by maintainers, not external contributors.

---

## Security

- Never commit API keys, tokens, or secrets
- Use environment variables and `.env` (git-ignored)
- `.secrets.baseline` in `scripts/detect-secrets/` tracks known-safe patterns
- Report security issues to security@anthropic.com (not via GitHub Issues)

---

## Resources

- **Anthropic Docs**: https://docs.claude.com
- **Models Overview**: https://docs.claude.com/en/docs/about-claude/models/overview
- **API Reference**: https://docs.claude.com/en/api/messages
- **Files API**: https://docs.claude.com/en/api/files-content
- **Skills Docs**: https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview
- **GitHub Issues**: https://github.com/anthropics/anthropic-cookbook/issues
- **Contributing Guide**: `CONTRIBUTING.md`
