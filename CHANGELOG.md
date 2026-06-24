# Changelog

All notable changes to this project are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Community health files: `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md`, issue
  templates, and a pull request template.
- `CHANGELOG.md` and an `/update-changelog` maintainer command for Claude Code.
- Generic `examples/notes-app/` worked config.

### Changed
- Rewrote the README and skill prose for a public launch: clearer voice, no em dashes.
- Decoupled the project from any specific app. The bundled example is now a generic `notes-app`.

## [0.1.0] - 2026-06-23

### Added
- Initial release of the naive-user plugin and skill.
- Source-blind, config-driven exploratory UX testing methodology in
  `skills/naive-user/SKILL.md`.
- `/naive-test` command for Claude Code, Codex, Gemini CLI, Copilot CLI, and OpenCode.
- Playwright MCP wiring for each supported harness.
- Config template, mental-model and findings templates, and a worked example config.

[Unreleased]: https://github.com/TechGardenCode/naive-user/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/TechGardenCode/naive-user/releases/tag/v0.1.0
