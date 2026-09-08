# Recommended GitHub repository settings

Some repository polish lives in GitHub settings rather than files. Apply this checklist when the project is ready for public contributions.

## General

- Description: `Dress full-screen macOS Spaces for instant recognition in Mission Control.`
- Website: leave unset until a project site/docs page exists.
- Suggested topics: `macos`, `swift`, `mission-control`, `spaces`, `fullscreen`, `desktop-switcher`, `accessibility`, `window-management`, `utility`.
- Enable Issues.
- Enable Discussions when there is enough traffic to separate support/design conversation from actionable issues.
- Keep Wiki disabled unless it gains a distinct purpose; versioned project knowledge belongs in `docs/`.

## Pull requests

Recommended:

- enable **squash merge**;
- enable **rebase merge** only if maintainers want to preserve contributor commit structure;
- disable merge commits once contribution volume grows, unless a concrete workflow needs them;
- automatically delete head branches after merge;
- allow updating PR branches when GitHub offers the setting.

Use concise squash commit subjects that make `main` history readable.

## Branch protection / ruleset for `main`

Once CI exists:

- require pull requests before merging;
- require at least one approval when there is more than one active maintainer;
- dismiss stale approvals when meaningful code changes;
- require conversation resolution;
- require passing checks;
- require linear history if merge commits are disabled;
- block force pushes and deletion;
- allow maintainer bypass only for security/repository recovery.

A single-maintainer project should avoid rules that make emergency maintenance impossible.

## Security

Enable where available:

- Private Vulnerability Reporting;
- Dependabot alerts;
- dependency graph;
- secret scanning / push protection for public repositories;
- CodeQL after meaningful source code exists and the signal is useful.

Do not collect security reports through public issues when private reporting is available.

## Releases

- Use GitHub Releases as the canonical public release record.
- Attach signed/notarized artifacts when distribution begins.
- Publish checksums.
- Keep release notes human-readable and link to the changelog.

## Labels

A small useful initial set:

### Type

- `type: bug`
- `type: feature`
- `type: research`
- `type: docs`
- `type: dsas`
- `type: security`

### Area

- `area: fullscreen`
- `area: split-screen`
- `area: mission-control`
- `area: identity`
- `area: styling`
- `area: platform`

### State

- `needs: reproduction`
- `needs: evidence`
- `needs: design`
- `blocked: macOS`
- `good first issue`
- `help wanted`

Avoid dozens of labels before issue volume justifies them.

## Milestones

Use product/risk milestones rather than calendar quarters initially:

1. `Feasibility`
2. `Single Full Screen`
3. `Split Full Screen`
4. `DSAS 0.1`
5. `First Public Release`

## Social preview

Once the visual prototype exists, create a 1280×640 social preview showing a row of nearly identical native dark thumbnails changing into clearly identified SpaceDress thumbnails. This communicates the problem faster than a logo alone.
