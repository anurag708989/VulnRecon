# Contributing to VulnRecon

Thanks for considering a contribution! This project is an open-source consolidation of several earlier prototype versions, so there's meaningful cleanup and hardening work available for contributors of any experience level.

## Ground Rules

1. **Responsible use only.** Do not add features that facilitate unauthorized scanning, or that hardcode targets/credentials. All examples in PRs must use placeholder domains (`example.com`) and placeholder keys.
2. **No silent failure by default.** New code should log errors (via the `logging` module) rather than swallowing exceptions with bare `except: pass`.
3. **Type hints required** for new functions/methods, consistent with the existing `dataclass`/`typing` usage.
4. **Keep the CLI backward compatible** where possible; deprecate flags with a warning before removing them.

## Getting Started

```bash
git clone https://github.com/<your-username>/vulnrecon.git
cd vulnrecon
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt -r requirements-dev.txt
pre-commit install   # if configured
```

## Good First Issues

- Replace bare `except: pass` blocks with proper logging (see `docs/ARCHITECTURE.md` → Error Handling Philosophy)
- Wire `--threads` into an `asyncio.Semaphore` to actually bound concurrency
- Add environment-variable support for `--shodan` / `--telegram` so secrets don't appear in shell history
- Add Shodan pagination + retry/backoff on HTTP 429
- Write the first `pytest` suite for `AdvancedPatternMatcher` scoring logic
- Add a GitHub Actions workflow: lint (`ruff`/`flake8`) + test (`pytest`) on every PR
- Add a `Dockerfile` bundling Python deps + Go tool binaries

See the full [Roadmap](README.md#roadmap) for larger initiatives.

## Development Workflow

1. Fork the repo and create a feature branch: `git checkout -b feat/shodan-pagination`
2. Make your changes with clear, atomic commits
3. Add/update tests under `tests/`
4. Run the test suite: `pytest -v`
5. Update relevant docs (`README.md`, `docs/FEATURES.md`, `docs/ARCHITECTURE.md`) if behavior changed
6. Open a PR against `main` describing **what** changed and **why**, referencing any related issue

## Commit Message Style

Conventional Commits preferred:
```
feat: add Shodan pagination support
fix: bound JS discovery concurrency with semaphore
docs: clarify FFUF/Nuclei target files are inputs, not auto-executed
```

## Reporting Bugs / Requesting Features

Use GitHub Issues with the provided templates (`.github/ISSUE_TEMPLATE/`). Include:
- VulnRecon version / commit hash
- Python version, OS
- Full command used (redact any real API keys/targets)
- Expected vs actual behavior, and logs if `--debug` was used

## Security Vulnerabilities in VulnRecon Itself

Do **not** open a public issue for security vulnerabilities in the tool itself (e.g., a flaw that could leak a user's own Shodan key). Instead, follow [`SECURITY.md`](SECURITY.md) for private disclosure.

## Code of Conduct

Be respectful, constructive, and assume good faith. Harassment or abusive behavior toward maintainers or other contributors will not be tolerated.
