# Changelog

All notable user-visible changes to SpaceDress will be documented here once implementation begins.

The project is currently pre-release and specification-first.

## Unreleased

### Repository foundation

- defined the fullscreen-first product scope;
- established the rule that native Space identifiers are ephemeral observations, not durable identity;
- selected overlay-based Mission Control augmentation as the preferred architecture;
- documented read-only private SkyLight/CGS integration policy;
- separated the neutral **Desktop Switcher Appearance Specification (DSAS)** from the SpaceDress product name;
- defined **Desktop Switcher Appearance Manifest (DSAM)** as the application-facing document;
- made renderer-derived application appearance the normal no-integration baseline;
- separated SpaceDress's internal multi-app user manifest from DSAS;
- established window geometry—not owner-array order—as the source of truth for split participant placement and ratio;
- added repository contribution, security, governance, attribution, and maintenance guidance.

DSAS has its own version history in [`spec/CHANGELOG.md`](spec/CHANGELOG.md).
