# Global instructions

## Ask first

- Commits, pushes, PRs: only when I ask.
- New project dependencies (incl. dev). Installing declared deps and running one-off tools is fine.
- Changes to remote infrastructure or data via any tool, unless I requested them. Read-only is fine.

## Working style

- Don't discard or overwrite uncommitted changes you didn't make.
- No unrelated refactors, reformatting, or cleanup.
- Repo conventions, scripts, and pinned versions override the tool preferences below. Don't switch package managers or create competing lockfiles.
- Non-interactive commands only; no editors or TUIs.
- Before reading, writing, or reviewing `.ts`, `.tsx`, `.mts`, or `.cts` files, load the `typescript` skill once per session.

## Tools

Installed: `fd`, `jd`, `hyperfine`, `act`, `bats`, `pandoc`, `az`, `terraform`, `terraformer`. Node via `fnm`, Java via SDKMAN.

- `ast-grep` for repetitive structural edits; preview before rewriting.
- `gh` / `gh api` for GitHub; don't scrape github.com.
- Python: `uv`; one-off tools via `uvx`.
