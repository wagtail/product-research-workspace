# Product research workspace for Wagtail

## What's included

| Concern                 | Tool                                                                               |
| ----------------------- | ---------------------------------------------------------------------------------- |
| Documentation site      | [MkDocs](https://www.mkdocs.org/) + Material for MkDocs, built strictly in CI      |
| Task runner             | [just](https://github.com/casey/just) (`justfile`)                                 |
| Python deps             | [uv](https://github.com/astral-sh/uv) (`pyproject.toml`)                           |
| Python lint/format      | [ruff](https://github.com/astral-sh/ruff)                                          |
| Type checking           | [mypy](https://mypy-lang.org/) + [ty](https://docs.astral.sh/ty/)                  |
| Formatting (non-Python) | [prettier](https://prettier.io)                                                    |
| Git hooks               | [prek](https://prek.j178.dev/) (`prek.toml`)                                       |
| Link checking           | [lychee](https://lychee.cli.rs) (`lychee.toml`)                                    |
| CI + deploy             | GitHub Actions: lint, strict docs build, Pages deploy (`.github/workflows/ci.yml`) |

## Site features

Configured in `mkdocs.yml` (each option commented inline):

- Light/dark themes with a custom local color palette (`docs/theme/theme.css`).
- Last-modified dates from git history per page.
- Tag index: `tags:` front matter collects pages at `/tags/`.
- `llms.txt` / `llms-full.txt` generation for LLM consumption.
- Abbreviation tooltips: `docs/contributing/abbreviations.md` is auto-appended to every page.
- Strict link and nav validation: pages must be in `nav` or opted out; broken anchors fail the build.

## Getting started

Requirements: `uv`, `just`, `prek`, `lychee`, Node.js (`node-version` pins the version).

```sh
just init      # Install dependencies, set up hooks
just docs      # Serve the docs locally on http://localhost:8001
just build-docs  # Strict build (what CI runs)
just lint      # ruff + mypy + ty + prettier checks
just format    # Auto-format
just check-links  # Link check (requires lychee)
```

## License

MIT — see [LICENSE](LICENSE).
