---
description: Drive the live app as a naive first-time user and report UX/behavior gaps before deploy
argument-hint: [app, defaults to the app in naive-user.config.json]
---

Run the **`naive-user-tester`** skill against the app named in `$ARGUMENTS`, or, if none is
given, the `app` in `naive-user.config.json`.

Invoke the skill and follow it exactly: read `naive-user.config.json`, make sure the app is
running, sign in via the configured auth steps, then explore the live UI as an uninformed
first-time user, **source-blind**, producing an updated mental model and a dated findings
report under `qa/naive-user/<app>/`.

This needs the Playwright MCP server (browser tools). See the project README if the `browser_*`
tools are not available. Run it on demand while developing, or dispatch it as a subagent to run
in parallel. It complements scripted end-to-end tests: those assert known flows, this discovers
unknown gaps.
