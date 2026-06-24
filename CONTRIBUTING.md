# Contributing

Thanks for your interest in improving naive-user-tester. Issues and pull requests are welcome.

## Ground rules

- Be respectful. This project follows the [Code of Conduct](CODE_OF_CONDUCT.md).
- For security issues, do not open a public issue. See [SECURITY.md](SECURITY.md).
- Keep changes focused. One concern per pull request is easier to review and ship.

## How the repo is laid out

Core content lives once, and each harness gets a thin manifest that points at it. When you
change behavior, you usually edit one of these two files:

- `skills/naive-user-tester/SKILL.md` is the source-blind testing methodology.
- `commands/naive-test.md` (plus `commands/naive-test.toml` for Codex and OpenCode) is the
  `/naive-test` entry point.

The harness manifests (`.claude-plugin/`, `.codex-plugin/`, `.github/plugin/`,
`gemini-extension.json`, `opencode.json`) only point at those files and declare the Playwright
MCP server. They should not duplicate methodology content.

## Making a change

1. Fork and branch from `main`.
2. Make your edit. If you change the methodology, keep it source-blind and config-driven, the
   two properties the whole project depends on.
3. If you touch the Playwright MCP wiring, update it in every harness manifest that declares it
   so the harnesses stay in sync.
4. Update [CHANGELOG.md](CHANGELOG.md) under `## [Unreleased]`. If you use Claude Code, the
   `/update-changelog` command in `.claude/commands/` does this for you from your staged diff.
5. Open a pull request using the template. Describe what a user will notice and which harnesses
   you tested against.

## Testing your change

There is no build step. To test end to end, point the plugin at a real running web app with a
`naive-user.config.json` and run `/naive-test`. Confirm the agent stays source-blind, signs in
via the configured auth steps, and writes a findings report under `qa/naive-user/<app>/`.

## Style

- Plain, direct prose. No em dashes in docs.
- Match the voice of the surrounding file.
- Markdown for docs, valid JSON or TOML for manifests.
