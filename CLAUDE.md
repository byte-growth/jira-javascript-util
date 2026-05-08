# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

A browser bookmarklet that injects a UI overlay into Jira issue pages to create subtasks in bulk from configurable team templates. All UI text is in Dutch. There is no build system, no package manager, and no test framework — the deliverable is a single JavaScript file intended to be saved as a browser bookmark with the `javascript:` protocol.

## Development workflow

No build step. Edit `subtaskcreator/Create Subtasks Jira.js` directly. To test, copy the file contents, create a browser bookmark prefixed with `javascript:`, and run it on a Jira issue page.

To minify/re-minify the script before committing, use any standard JS minifier (e.g., `terser` CLI or an online tool). The file should remain a single line for bookmarklet use.

## Architecture

Everything lives in a single IIFE in `subtaskcreator/Create Subtasks Jira.js`. Key internal concepts:

- **Config**: Stored in `localStorage` under key `__jst_config` as JSON. Schema: `{ teams: { [teamName]: { [issueType]: string[] } } }` where each string is a subtask summary prefixed with `- `.
- **Overlay**: Injected DOM element with id `__jst_ov`. Guards against double-injection.
- **Jira integration**: Hardcoded base URL `https://jira.om.minjenv.nl`. Uses `POST /rest/api/2/issue` to create subtasks and `GET /rest/api/2/issue/{key}?fields=assignee` to optionally inherit the parent's assignee. Authentication is cookie-based (`credentials: 'include'`) with `X-Atlassian-Token: no-check`.
- **Issue key detection**: Parsed from `window.location` or the page title.
- **Project key**: Derived from the issue key (everything before the first `-`).
