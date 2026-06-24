# naive-user-tester

Drive your **live web app as an uninformed, source-blind first-time user** — hover, click,
type in a real browser, observe what actually happens, and report the gaps (bugs, broken
expectations, UX surprises, a11y) before real users hit them.

It fills the gap between three things that *don't* catch first-impression UX problems:

- **Real users** — find these gaps, but only after you ship.
- **You, manually** — you know how it's built, so you can't see it fresh.
- **Scripted e2e tests** — assert *known* flows; they can't discover the *unknown* ones.

The skill forms expectations from only two sources — **what's on the screen** and
**universal web conventions** — never from your source code. It builds a knowledge base
(`qa/naive-user/<app>/`) that lives *outside* the code, the way a real user's mental model
does, and compounds across runs.

## What you get

```
qa/naive-user/<app>/
├── mental-model.md        # what a naive user believes the app does; refined every run
├── findings/<date>.md     # dated gap reports (Expected / Did / Observed / Gap / Severity / Repro)
└── screenshots/           # before/after evidence (gitignored)
```

Findings are severity-ranked: `bug`, `broken-expectation`, `ux-gap`, `surprise`, `a11y`.

## Requirements

- **Node.js 18+** — the only hard requirement. The Playwright MCP server provisions its
  browser on first use. (If you ever hit a missing-browser error, run `npx playwright install chromium`.)
- **A running app** — already serving at a URL, or a `startCommand` in your config.
- **One of the supported harnesses** below (each needs agentic tool-calling **and** the
  Playwright MCP browser tools — that's why instruction-only IDE rule hosts aren't targeted).

## Install

### 1. Add the plugin/skill to your harness *and* wire Playwright MCP

The same MCP server body — `npx @playwright/mcp@latest` — works everywhere; only OpenCode and
Copilot tweak the shape.

**Claude Code** — marketplace install; the plugin bundles the Playwright MCP, so it's one step:

```text
/plugin marketplace add TechGardenCode/naive-user-tester      # or a local path: ./naive-user-tester
/plugin install naive-user-tester@naive-user-tester
```

> Not using the plugin? Copy this repo's `.mcp.json` into your app repo root and drop
> `skills/` + `commands/` into your project's `.claude/`.

**Codex** — drop the plugin in, then add the MCP server to `~/.codex/config.toml`:

```toml
[mcp_servers.playwright]
command = "npx"
args = ["@playwright/mcp@latest"]
```

**Gemini CLI** — install as an extension (`gemini-extension.json` bundles both the MCP server
and the skill as context), or add the server to `~/.gemini/settings.json`:

```json
{
  "mcpServers": {
    "playwright": { "command": "npx", "args": ["@playwright/mcp@latest"] }
  }
}
```

**GitHub Copilot CLI** — plugin lives at `.github/plugin/plugin.json` (and bundles the MCP).
To wire it manually, add to `~/.copilot/mcp-config.json`:

```json
{
  "mcpServers": {
    "playwright": { "type": "local", "command": "npx", "args": ["@playwright/mcp@latest"], "tools": ["*"] }
  }
}
```

**OpenCode** — `opencode.json` bundles the MCP (note: `command` is an array, and `-y` avoids
the interactive npx prompt):

```json
{
  "mcp": {
    "playwright": { "type": "local", "command": ["npx", "-y", "@playwright/mcp@latest"], "enabled": true }
  }
}
```

### 2. Point it at your app

Copy `templates/naive-user.config.json` into **your app repo's root** and fill it in:

```json
{
  "app": "myapp",
  "baseUrl": "http://localhost:3000",
  "startCommand": null,
  "auth": { "steps": ["Go to /", "Type the dev username", "Submit"], "critiqueLoginPage": false },
  "coverageNotes": "Optional hints, not a script."
}
```

`startCommand` replaces any "how do I start the app" step — set it and the skill runs it; leave
it `null` and the skill assumes the app is already up at `baseUrl`. See `examples/iris/` for a
fully worked config.

### 3. Keep the knowledge base reviewable

In your **app repo's** `.gitignore`, commit the markdown but ignore the screenshot evidence:

```gitignore
qa/naive-user/*/screenshots/
```

## Usage

```text
/naive-test [app]
```

With no argument it uses the `app` from `naive-user.config.json`. The skill loads the prior
mental model, makes sure the app is running, signs in via the configured auth steps, explores
the live UI source-blind, and writes an updated `mental-model.md` plus a dated findings report.
Run it on demand while developing, or dispatch it as a subagent to run in parallel with other work.

A changed-from-last-time behavior is flagged as a **regression**.

## How it ships (single source → thin adapters)

Core content lives **once**:

- `skills/naive-user-tester/SKILL.md` — the source-blind testing methodology (config-driven).
- `commands/naive-test.md` (+ `.toml` for Codex/OpenCode) — the `/naive-test` entry point.

Each harness gets a thin manifest that *points at* those files and declares the Playwright MCP
in that harness's native format — no content is duplicated:

| Harness | Manifest | MCP declared in |
|---|---|---|
| Claude Code | `.claude-plugin/plugin.json` (+ `marketplace.json`) | plugin `mcpServers` / `.mcp.json` |
| Codex | `.codex-plugin/plugin.json` | `~/.codex/config.toml` |
| Gemini CLI | `gemini-extension.json` | extension `mcpServers` / `settings.json` |
| Copilot CLI | `.github/plugin/plugin.json` | plugin `mcpServers` / `~/.copilot/mcp-config.json` |
| OpenCode | `opencode.json` | `opencode.json` `mcp` |

## License

MIT — see [LICENSE](LICENSE).
