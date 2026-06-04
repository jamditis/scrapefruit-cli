# Repository Guidelines

## Project structure and module organization

This directory is primarily a Node-based project. Start with `lib/`, `resources/`. Keep new code, docs, and assets beside the feature they support instead of adding another generic catch-all folder.

## Build, test, and development commands

- `npm install` installs dependencies.

## Coding style and naming conventions

- Use `PascalCase` for components and classes, `camelCase` for functions and utilities, and kebab-case for non-code asset filenames unless the repo already differs.
- Match the existing formatter and linter; most Node and frontend repos here use 2-space indentation.
- Read `CLAUDE.md` before larger edits; it carries repo-specific constraints that override generic habits.

## Testing guidelines

- No formal test harness is obvious at the repo root. Add targeted automated checks when you touch executable code, and record manual verification in the PR or task notes.

## Commit and pull request guidelines

- Recent history favors short, imperative subjects. Where it fits, use prefixes like `feat:`, `fix:`, `docs:`, `refactor:`, or `chore:` and keep the first line under about 72 characters.
- PRs should explain the why, link the issue or task when one exists, and include screenshots or sample payloads for UI, API, or report changes.
