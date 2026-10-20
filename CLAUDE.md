# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A workspace for browser automation (testing web apps, taking screenshots) driven through **Playwright CLI** (`@playwright/cli`, installed globally). There is no application code, build, lint, or test suite; work is done by issuing `playwright-cli` commands. The project skill at `.claude/skills/playwright-cli/SKILL.md` (plus `references/`) documents the full command set — load it before browser work. `playwright-cli --help <command>` is the authoritative source for flags.

## Environment (Windows, nvm-windows)

- `playwright-cli` requires Node 18+ and is installed under **Node 24.22.0**. Global npm packages are per-Node-version, so it disappears if another Node version is active.
- `nvm use` is broken on this machine (nvm-windows 1.2.2 fails on the space in the user path `C:\Users\Basavachetan GM`). Switch Node by repointing the junction instead (this is system-wide):
  ```powershell
  Remove-Item C:\nvm4w\nodejs; New-Item -ItemType Junction -Path C:\nvm4w\nodejs -Target "$env:LOCALAPPDATA\nvm\v24.22.0"
  ```
- The default browser is the installed Chrome (set by `playwright-cli install`). To download a bundled browser use `playwright-cli install-browser`, not `npx playwright install` (that pulls a separately-versioned `playwright` package).

## Typical workflow

```powershell
playwright-cli open https://example.com      # headless by default; --headed to watch
playwright-cli snapshot                      # accessibility tree with element refs (e1, e2, ...)
playwright-cli click e7                      # act on refs from the latest snapshot
playwright-cli screenshot --filename=screenshots/name.png   # add --full-page for the whole page
playwright-cli close
```

- Refs come from the most recent snapshot — re-snapshot after navigation or DOM changes rather than reusing old refs or hardcoding link text.
- `--filename` paths resolve relative to the project root.
- In PowerShell, piping output into `Select-Object -First N` truncates the pipe and yields exit code 255 even when the command succeeded.

## Layout

- `screenshots/` — saved screenshots.
- `.playwright-cli/` — auto-generated page snapshots (`page-<timestamp>.yml`) written on each `open`/navigation; disposable.
- `.playwright/` — workspace config dir; `cli.config.json` here (if created) is picked up by `open --config` by default.
