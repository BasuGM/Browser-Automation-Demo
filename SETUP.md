# Project setup

This repo is a workspace for driving a real browser from Claude Code using **Playwright CLI** (`@playwright/cli`). It includes a demo site, `site/index.html`, which is a one-question-per-page onboarding form. Claude Code built it and tested it end to end with `playwright-cli`.

This guide covers:

1. [Prerequisites](#1-prerequisites)
2. [Install Claude Code](#2-install-claude-code)
3. [Install Playwright CLI](#3-install-playwright-cli)
4. [Connect Playwright CLI to Claude Code (the skill)](#4-connect-playwright-cli-to-claude-code)
5. [Run the demo site](#5-run-the-demo-site)
6. [Verify everything works](#6-verify-everything-works)
7. [Troubleshooting](#7-troubleshooting)

---

## Project layout

```
.
├── CLAUDE.md                          # Project instructions Claude Code reads automatically
├── SETUP.md                           # This file
├── site/
│   └── index.html                     # Demo onboarding form (single file, no build step)
├── screenshots/                       # Screenshots saved by playwright-cli
├── .claude/
│   └── skills/playwright-cli/         # Skill that teaches Claude Code the playwright-cli commands
│       ├── SKILL.md
│       └── references/
└── .playwright-cli/                   # Auto-generated page snapshots and console logs (disposable)
```

---

## 1. Prerequisites

| Requirement | Why |
|---|---|
| **Node.js 18+** (tested on 24.22.0) | `playwright-cli` and `npx serve` run on Node |
| **Google Chrome** | Default browser `playwright-cli` drives (a bundled Chromium can be used instead) |
| **A Claude account** (Pro, Max, Team, Enterprise, or API) | To sign in to Claude Code |
| **Git for Windows** (Windows only) | Claude Code on Windows uses Git Bash for shell commands |

### macOS: install Node

Pick one option.

```bash
# Option A: Homebrew
brew install node

# Option B: nvm (lets you switch versions)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
# restart the terminal, then:
nvm install 24
nvm use 24
```

### Windows: install Node

Pick one option.

```powershell
# Option A: winget
winget install OpenJS.NodeJS.LTS

# Option B: nvm-windows (https://github.com/coreybutler/nvm-windows/releases)
nvm install 24.22.0
nvm use 24.22.0
```

> **nvm-windows and spaces in your user path.** If your Windows username contains a space (e.g. `C:\Users\Jane Doe`), `nvm use` can fail. You can repoint the active-Node junction manually instead. This applies system-wide. If the command fails with access denied, run it from an elevated PowerShell:
>
> ```powershell
> Remove-Item C:\nvm4w\nodejs
> New-Item -ItemType Junction -Path C:\nvm4w\nodejs -Target "$env:LOCALAPPDATA\nvm\v24.22.0"
> ```
>
> Check where your nvm symlink lives with `nvm root` and adjust `C:\nvm4w\nodejs` if it differs.

Check the installed version on either OS:

```bash
node -v   # should print v18 or higher
```

---

## 2. Install Claude Code

### macOS

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

### Windows (PowerShell)

```powershell
irm https://claude.ai/install.ps1 | iex
```

**Alternative on either OS:** install through npm.

```bash
npm install -g @anthropic-ai/claude-code
```

Then sign in from the project folder:

```bash
cd path/to/Browser-Automation-Demo
claude
```

On first run, Claude Code opens a browser window to log in. If you work in VS Code, you can also install the **Claude Code** extension from the marketplace and open this folder.

---

## 3. Install Playwright CLI

The commands are identical on macOS and Windows.

```bash
npm install -g @playwright/cli@latest
playwright-cli --version     # this repo was set up with 0.1.22
```

Initialize the workspace from the project root. This sets the default browser to your installed Chrome:

```bash
playwright-cli install
```

If you don't have Chrome, or want a bundled browser instead:

```bash
playwright-cli install-browser            # downloads Playwright's bundled browser
playwright-cli install-browser --list     # shows what is installed
```

> Use `playwright-cli install-browser`, **not** `npx playwright install`. The latter pulls in a separately versioned `playwright` package that the CLI won't use.

> **Global packages are per Node version.** If you switch Node versions with nvm (or nvm-windows), `playwright-cli` "disappears" until you switch back or reinstall it under the new version.

---

## 4. Connect Playwright CLI to Claude Code

Claude Code learns the `playwright-cli` command set from a **skill**: a `SKILL.md` file plus reference docs under `.claude/skills/`.

**This repo already includes the skill** at `.claude/skills/playwright-cli/`, so if you cloned it, there is nothing to do. Claude Code picks the skill up automatically when you start `claude` in this folder.

To set up the skill in a **new** project, or to refresh it after upgrading `@playwright/cli`:

```bash
# Install into this project's .claude/skills/
playwright-cli install --skills

# Or install once for all projects, into ~/.claude/skills/
playwright-cli install --skills --global
```

### Confirm Claude Code sees the skill

Start Claude Code in the project and type `/`. **playwright-cli** should appear in the list of skills. You can also just ask:

> Use playwright-cli to open https://example.com and take a screenshot.

The first time Claude Code runs a `playwright-cli` command, it asks for permission. Choose **"Yes, and don't ask again"** for `playwright-cli` commands to skip the prompt next time. This saves the rule to `.claude/settings.local.json`.

### Shells per OS

| | macOS | Windows |
|---|---|---|
| Terminal Claude Code uses | zsh/bash | PowerShell and Git Bash |
| Quoting URLs with `&` | `"https://x.com/?a=1&b=2"` | PowerShell: `playwright-cli --% goto "https://x.com/?a=1&b=2"` |
| Pitfall | none | Piping into `Select-Object -First N` returns exit code 255 even on success |

---

## 5. Run the demo site

The demo site is a single HTML file with no dependencies. Serve it locally on port 8080:

```bash
npx serve site -l 8080
```

The first run downloads the `serve` package through npx. After that, open:

- **http://localhost:8080/?reset**: a clean start. `?reset` clears any saved answers.
- **http://localhost:8080**: resumes where you left off. Answers are saved in your browser's localStorage.

You can also open `site/index.html` directly in a browser by double-clicking it.

**What the form does:** it asks 12 questions, one per screen. Press Enter to advance and ↑ to go back. On choice questions, the letter keys (A, B, C…) pick an option and advance. The final summary has Edit links, a JSON download, and Start over. No data leaves the browser.

---

## 6. Verify everything works

Run this smoke test by hand, or ask Claude Code to do it. Keep the server from step 5 running in another terminal.

```bash
playwright-cli open "http://localhost:8080/?reset" --headed   # --headed shows the browser window
playwright-cli snapshot                                       # accessibility tree with refs (e1, e2, ...)
playwright-cli fill "#q-field" "Ada"
playwright-cli press Enter
playwright-cli snapshot                                       # should now show the last-name question
playwright-cli screenshot --filename=screenshots/smoke.png
playwright-cli close
```

**Expected result:** the second snapshot shows *"Nice to meet you, Ada. What's your last name?"*, and `screenshots/smoke.png` exists.

Useful habits when driving the browser:

- **Re-snapshot after every navigation or screen change.** Element refs (`e7`, `f1e19`, …) come from the latest snapshot only.
- **Don't add sleeps.** `playwright-cli` has no wait command. To wait for something, poll with `snapshot` or `eval` until it appears. On this form, each screen sets `data-ready="true"` once its transition finishes:
  ```bash
  playwright-cli --raw eval "document.querySelector('.screen').dataset.ready"
  ```
- `--filename` paths are relative to the project root.

Or just ask Claude Code something like:

> Start the server, then use playwright-cli to fill out the whole onboarding form with test data and screenshot the summary page.

---

## 7. Troubleshooting

| Symptom | Fix |
|---|---|
| `playwright-cli: command not found` | You're on a different Node version than the one you installed it under. Run `node -v`, switch back, or reinstall with `npm install -g @playwright/cli@latest`. |
| `nvm use` fails on Windows with a path error | Your user path has a space. Use the junction workaround in [section 1](#windows-install-node). |
| `net::ERR_CONNECTION_REFUSED` on `open` | The server isn't up yet. Start `npx serve site -l 8080` and wait for it to print the address. |
| The browser opens but nothing is visible | The browser is headless by default. Add `--headed`. |
| Stale or "ref not found" errors | Run `playwright-cli snapshot` again and use the new refs. |
| Leftover browser processes | `playwright-cli close-all`, or `playwright-cli kill-all` as a last resort. |
| The form opens mid-flow or on the summary | It is resuming saved answers. Add `?reset` to the URL, or click **Start over**. |
| Claude Code doesn't use playwright-cli | Check that `.claude/skills/playwright-cli/SKILL.md` exists (run `playwright-cli install --skills`), then restart `claude`. |

For any command's options, run `playwright-cli --help <command>`. That output is the authoritative reference.
