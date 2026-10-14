# Global instructions

## Working style

- Do not commit, push, or open PRs unless I ask.
- Ask before adding new dependencies.

## CLI tools available

- Search: `rg` for content, `fd` for files, `tree -L 2` for directory layout. Prefer them over `grep -r` and `find`.
- Structural code search and rewrite: `ast-grep`, for example `ast-grep -p 'console.log($$$)' -l ts`. Prefer it over regex for refactors such as renaming calls or changing function signatures. Preview matches before applying a rewrite with `-r ... -U`.
- JSON: `jq` for querying, `jd` for structural diffs.
- Benchmarking: `hyperfine` to compare command or script performance, for example `hyperfine --warmup 3 'cmd-a' 'cmd-b'`.
- GitHub: use `gh` for PRs, issues, Actions runs and logs, and `gh api` for anything else. Do not scrape github.com.
- CI: validate workflows with `actionlint`; run them locally with `act` when useful.
- Shell scripts: check with `shellcheck`; test with `bats`.
- JS/TS: use the project's package manager, detected from the lockfile (`pnpm`, `yarn`, `bun`, or `npm`). Node is managed by `fnm`. Use project-local formatters and linters, not global ones.
- Python: use `uv` (`uv run`, `uv add`); run one-off tools through `uvx`, for example `uvx ruff check`.
- Go: `go test ./...`, `gofmt`, `gopls`.
- Java via SDKMAN.
- Docker is available.
- Documents: `pandoc` for format conversion.

## Safety

- `kubectl`, `aws`, `az`, `terraform`, and `terraformer` are installed. Run only read-only commands; ask before anything that changes infrastructure (apply, delete, deploy, scale, destroy).
- Do not run interactive programs such as `lazygit`, `nvim`, `vim`, `fzf`, or `code`.
