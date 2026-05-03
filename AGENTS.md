# AGENTS.md

## Repo status

Python package with CLI scaffolding in place. `pyproject.toml` exists with `click` as runtime dep, `ruff` and `pytest` as dev deps under `[project.optional-dependencies]`. CLI entry point at `src/wazuh_decoder_gen/cli.py` exposes `--help` and `--version`. One smoke test passes. No CI pipeline yet.

Layout: `src/wazuh_decoder_gen/` (source), `tests/` (tests), `fixtures/` (sample logs & XML).

## Toolchain (all enforced by agents)

| Tool | Role |
|------|------|
| `ruff` | Linting + formatting (`ruff check src tests`, `ruff format .`) |
| `pytest` | Testing (`pytest -x`) |
| Python ≥3.11 | Type hints mandatory for public functions |

No pre-commit hooks or CI pipeline exist yet — agents should not assume any automated gating.

## Commands that matter

```bash
# Activate the venv first (created by issue #1)
source .venv/bin/activate

# Full verification (run after every change)
ruff check src tests && ruff format --check src tests && pytest -x

# Single test
pytest tests/path/to/test_file.py -v

# Single test class or function
pytest tests/path/to/test_file.py::TestClass::test_method -v

# Interactive debugging
python -m pytest tests/path/to/test_file.py -v -s
```

**Never skip the full verification step.** Tests + linting must both pass before declaring a task done.

## File conventions

- Source under `src/wazuh_decoder_gen/`
- Tests mirror source structure under `tests/`
- Sample logs and expected Wazuh XML go in `fixtures/` (never hardcode XML in source)
- CLI entry point: `src/wazuh_decoder_gen/cli.py`
- Public API functions require type hints; private helpers may omit when obvious

## Domain: Wazuh decoders & rules

- **Decoders** parse log lines into named fields (`program_name`, `srcip`, `dstport`, etc.) using PCRE-compatible regex
- **Rules** match decoded fields, assign severity (level 0–15), and define signature IDs
- **Local rules** must use IDs ≥ 100000 to avoid conflicts with Wazuh's built-in rules
- **Parent/child decoder hierarchy** matters — child decoders are invoked by parent decoders and their output is available to rules
- Generated output should be valid Wazuh XML with proper nesting (`decoder`, `rule`, `group` elements)

## Working pattern (orchestrator + coder)

This repo uses a dual-agent workflow configured in `.opencode/agents/`:

- `orchestrator` (`.opencode/agents/orchestrator.md`) — plans and delegates; **cannot edit files**
- `coder` (`.opencode/agents/coder.md`) — implements precise subtasks; **can edit and run commands**

For GitHub issues: fetch via `gh issue view N --json title,body` before starting work.

## Keeping this file accurate

AGENTS.md must reflect current repo reality. When agents make material changes — new dependencies, new module structure, new conventions, new CI, new toolchain — they must update this file as part of the same task. Stale AGENTS.md is worse than no AGENTS.md.
