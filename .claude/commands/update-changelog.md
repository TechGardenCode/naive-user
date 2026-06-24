---
description: Update CHANGELOG.md [Unreleased] from the current git diff (maintainer tool)
argument-hint: [optional, a version to cut, e.g. 0.2.0]
---

Keep `CHANGELOG.md` current. This is a maintainer tool for developing this repo, not part of
the shipped plugin.

1. Run `git diff HEAD` and `git status` to see what changed since the last commit. If nothing
   is staged or modified, also look at commits since the latest tag (`git log $(git describe
   --tags --abbrev=0 2>/dev/null)..HEAD --oneline`).
2. Summarize the changes a **user of the plugin** would notice, not the mechanics. Sort each
   entry under the right [Keep a Changelog](https://keepachangelog.com/) heading: `Added`,
   `Changed`, `Deprecated`, `Removed`, `Fixed`, `Security`.
3. Edit `CHANGELOG.md`: merge those entries into the `## [Unreleased]` section. Do not
   duplicate an entry that is already there. Keep the prose plain, no em dashes.
4. If `$ARGUMENTS` names a version, cut a release: rename `## [Unreleased]` to
   `## [<version>] - <today's date in YYYY-MM-DD>`, add a fresh empty `## [Unreleased]` above
   it, bump the `version` field in every plugin manifest (`.claude-plugin/plugin.json`,
   `.codex-plugin/plugin.json`, `.github/plugin/plugin.json`, `gemini-extension.json`), and
   update the compare links at the bottom of the changelog.
5. Show the diff you made. Do not commit unless asked.
