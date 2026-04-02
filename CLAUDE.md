# ctx — Plugin Development

This is a Claude Code plugin. Changes here affect all workspaces that install it.

## Releasing

Before pushing to `main`, always:

1. **Bump the version** in `.claude-plugin/plugin.json` following semver:
   - Patch (`x.x.+1`): bug fixes, wording tweaks
   - Minor (`x.+1.0`): new skills, new routing rules, structural changes
   - Major (`+1.0.0`): breaking changes to skill behavior or workspace format
2. **Commit the version bump** together with the changes (or as a separate commit — either is fine).

Do not push without bumping the version. Users won't receive updates unless the version changes.

## Structure

- `skills/` — Each subdirectory is a skill with a `SKILL.md` file.
- `agents/` — Agent definitions.
- `.claude-plugin/plugin.json` — Plugin metadata and version.
- `.claude-plugin/marketplace.json` — Marketplace listing.

## Key rule

Templates (CLAUDE.md, _ctx/CLAUDE.md, _template/*) are defined **only** in `skills/init/SKILL.md`. Other skills that need them (like `update`) read from init at runtime — never duplicate templates.
