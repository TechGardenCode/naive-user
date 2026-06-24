# Security Policy

## Reporting a vulnerability

Please do not open a public issue for security problems.

Report vulnerabilities privately through GitHub's
[Report a vulnerability](https://github.com/TechGardenCode/naive-user-tester/security/advisories/new)
flow, or by email to **techgardencode@gmail.com**. Include a description, steps to reproduce,
and the impact you expect. You will get an acknowledgement within a few days, and a fix or
mitigation plan once the report is confirmed.

## Scope

This project ships prompt and manifest files. It does not run a service. The main security
surface to keep in mind:

- The plugin wires up the [Playwright MCP server](https://github.com/microsoft/playwright-mcp),
  which drives a real browser against the URL in your `naive-user.config.json`. Only point it at
  apps and environments you control.
- `startCommand` in a config is executed by your harness to launch the app under test. Review
  any config you did not write before running it, the same as any script in a repo.

Issues in the Playwright MCP server or in a harness itself should be reported to those projects.
This policy covers the content in this repository.

## Supported versions

This project is pre-1.0. Fixes land on `main` and ship in the next tagged release.
