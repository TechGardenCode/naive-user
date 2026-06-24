# naive-user-tester

Drive your live web app as a source-blind, first-time user. An AI agent hovers,
clicks, and types in a real browser, watches what actually happens, and reports the
gaps (bugs, broken expectations, UX surprises, accessibility issues) before real users
hit them.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Harnesses](https://img.shields.io/badge/harnesses-Claude%20Code%20%7C%20Codex%20%7C%20Gemini%20%7C%20Copilot%20%7C%20OpenCode-444)](#install)
[![Powered by Playwright MCP](https://img.shields.io/badge/powered%20by-Playwright%20MCP-2EAD33)](https://github.com/microsoft/playwright-mcp)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

It covers a gap that three other things miss:

- **Real users** find these problems, but only after you ship.
- **You, testing manually,** know how the app is built, so you cannot see it fresh.
- **Scripted end-to-end tests** assert flows you already know. They cannot discover the ones you do not.

The agent forms expectations from only two sources: what is on the screen, and
universal web conventions. It never reads your source code. It builds a knowledge base
under `qa/naive-user/<app>/` that lives outside the code, the way a real user's mental
model does, and it compounds across runs.

<!-- Add a short demo GIF or screenshot here before launch, then set a social preview
     image in Settings > General > Social preview. Both noticeably lift first impressions. -->

## Contents

- [What you get](#what-you-get)
- [Requirements](#requirements)
- [Install](#install)
- [Configure](#configure)
- [Usage](#usage)
- [Example output](#example-output)
- [How it ships](#how-it-ships)
- [Contributing](#contributing)
- [License](#license)

## What you get

```
qa/naive-user/<app>/
├── mental-model.md        # what a naive user believes the app does; refined every run
├── findings/<date>.md     # dated gap reports (Expected / Did / Observed / Gap / Severity / Repro)
└── screenshots/           # before/after evidence (gitignored)
```

Findings are severity-ranked: `bug`, `broken-expectation`, `ux-gap`, `surprise`, `a11y`.

## Requirements

- **Node.js 18+** is the only hard requirement. The Playwright MCP server provisions its
  browser on first use. If you ever hit a missing-browser error, run `npx playwright install chromium`.
- **A running app**, either already serving at a URL or startable from a `startCommand` in your config.
- **One of the supported harnesses** below. Each needs agentic tool-calling plus the
  Playwright MCP browser tools, which is why instruction-only IDE rule hosts are not targeted.

## Install

### 1. Add the plugin to your harness and wire Playwright MCP

The same MCP server body, `npx @playwright/mcp@latest`, works everywhere. Only OpenCode
and Copilot tweak the shape.

**Claude Code.** Marketplace install. The plugin bundles the Playwright MCP, so it is one step:

```text
/plugin marketplace add TechGardenCode/naive-user-tester      # or a local path: ./naive-user-tester
/plugin install naive-user-tester@naive-user-tester
```

> Not using the plugin? Copy this repo's `.mcp.json` into your app repo root and drop
> `skills/` and `commands/` into your project's `.claude/`.

**Codex.** Drop the plugin in, then add the MCP server to `~/.codex/config.toml`:

```toml
[mcp_servers.playwright]
command = "npx"
args = ["@playwright/mcp@latest"]
```

**Gemini CLI.** Install as an extension (`gemini-extension.json` bundles both the MCP server
and the skill as context), or add the server to `~/.gemini/settings.json`:

```json
{
  "mcpServers": {
    "playwright": { "command": "npx", "args": ["@playwright/mcp@latest"] }
  }
}
```

**GitHub Copilot CLI.** The plugin lives at `.github/plugin/plugin.json` and bundles the MCP.
To wire it manually, add to `~/.copilot/mcp-config.json`:

```json
{
  "mcpServers": {
    "playwright": { "type": "local", "command": "npx", "args": ["@playwright/mcp@latest"], "tools": ["*"] }
  }
}
```

**OpenCode.** `opencode.json` bundles the MCP. Note that `command` is an array, and `-y`
avoids the interactive npx prompt:

```json
{
  "mcp": {
    "playwright": { "type": "local", "command": ["npx", "-y", "@playwright/mcp@latest"], "enabled": true }
  }
}
```

## Configure

### 1. Point it at your app

Copy `templates/naive-user.config.json` into your app repo's root and fill it in:

```json
{
  "app": "myapp",
  "baseUrl": "http://localhost:3000",
  "startCommand": null,
  "auth": { "steps": ["Go to /", "Type the dev username", "Submit"], "critiqueLoginPage": false },
  "coverageNotes": "Optional hints, not a script."
}
```

`startCommand` replaces any "how do I start the app" step. Set it and the agent runs it.
Leave it `null` and the agent assumes the app is already up at `baseUrl`. See
[`examples/notes-app/`](examples/notes-app/) for a fully worked config.

### 2. Keep the knowledge base reviewable

In your app repo's `.gitignore`, commit the markdown but ignore the screenshot evidence:

```gitignore
qa/naive-user/*/screenshots/
```

## Usage

```text
/naive-test [app]
```

With no argument it uses the `app` from `naive-user.config.json`. The agent loads the prior
mental model, makes sure the app is running, signs in via the configured auth steps, explores
the live UI source-blind, and writes an updated `mental-model.md` plus a dated findings report.
Run it on demand while developing, or dispatch it as a subagent to run in parallel with other work.

A changed-from-last-time behavior is flagged as a **regression**.

## Example output

A findings file leads with a one-line summary and a severity-sorted table, then one entry per gap:

```markdown
| # | Severity | Surface | Gap |
|---|----------|---------|-----|
| 1 | bug      | Capture | Pressing Enter in the title field reloads the page |
| 2 | a11y     | Sidebar | Active nav item has no visible focus ring |

## 1. Pressing Enter reloads instead of saving
- Expected: Enter submits the form (primary-button convention).
- Did: Typed a title, pressed Enter.
- Observed: Full page reload, draft lost.
- Severity: bug
- Repro: 1. Open /. 2. Type in title. 3. Press Enter.
- Screenshot: screenshots/capture-enter-before.png
```

## How it ships

Core content lives once:

- `skills/naive-user-tester/SKILL.md` holds the source-blind testing methodology (config-driven).
- `commands/naive-test.md` (plus `.toml` for Codex and OpenCode) is the `/naive-test` entry point.

Each harness gets a thin manifest that points at those files and declares the Playwright MCP
in that harness's native format. No content is duplicated:

| Harness | Manifest | MCP declared in |
|---|---|---|
| Claude Code | `.claude-plugin/plugin.json` (plus `marketplace.json`) | plugin `mcpServers` / `.mcp.json` |
| Codex | `.codex-plugin/plugin.json` | `~/.codex/config.toml` |
| Gemini CLI | `gemini-extension.json` | extension `mcpServers` / `settings.json` |
| Copilot CLI | `.github/plugin/plugin.json` | plugin `mcpServers` / `~/.copilot/mcp-config.json` |
| OpenCode | `opencode.json` | `opencode.json` `mcp` |

## Contributing

Issues and pull requests are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for the workflow,
and [SECURITY.md](SECURITY.md) to report a vulnerability privately.

## License

MIT. See [LICENSE](LICENSE).
