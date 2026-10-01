---
name: odoo-migration
description: End-to-end OCA module migration workflow for porting addons between Odoo major versions and preparing [MIG] pull requests. Use when migrating OCA modules, porting addons between Odoo versions, applying OCA migration checklists, resolving migration patch conflicts, or preparing versioned OCA migration PRs.
---

# Odoo Migration

Use this skill for OCA module migration between Odoo major versions. Compose
with `odoo-report-migration` when the module contains QWeb/PDF reports whose
visual output must be preserved.

This is not an OpenUpgrade skill. Do not adopt the OpenUpgrade operating model, its commands, or its workflow for a normal OCA `[MIG]` port. Consulting OpenUpgrade's `upgrade_analysis.txt` for core-module data-model changes IS part of the OCA protocol, because the version guides deliberately cover framework changes only and point there for data-model deltas. If a migration requires project-specific data handling, inspect the addon and repo conventions directly.

## First Move

Identify:

- source Odoo version
- target Odoo version, including Odoo 20.0 when the target branch is available
- OCA repository name
- module path/name
- current branch and remotes
- whether the corresponding OCA migration issue or PR context is known

If the Odoo framework source or local development setup is needed and not inside the workspace, ask the user to set `ODOO_SOURCE=/path/to/odoo`, `ODOO_BASE_COMMAND="..."`, and `ODOO_TOOL_README=/path/to/README.md` as needed.

## Workflow

Use [OCA-MODULE-MIGRATION.md](OCA-MODULE-MIGRATION.md) as the end-to-end guide.

Always apply every intermediate version checklist for jumps across more than one major version. Examples: migrating `15.0` to `18.0` requires the `16.0`, `17.0`, and `18.0` sections; migrating `18.0` to `20.0` requires the `19.0` and `20.0` sections.

Before creating a migration branch, verify that the OCA repository exposes the target branch and that the related migration issue or PR context is known. Odoo 20 support in this skill remains the normal OCA migration workflow, not OpenUpgrade.

The OCA protocol opens with a preamble: refresh the OCA conventions, subscribe to the project's mailing list, announce the module(s) on the `Migration to version <target>` issue, and install `pre-commit`. The issue announcement is the documented OCA step.

The migration is not done at the final commit. It reaches OCA when the branch is pushed to the contributor's fork and the pull request is opened with the `[<target>][MIG] <module>: Migration to <target>` title. See [OCA-MODULE-MIGRATION.md](OCA-MODULE-MIGRATION.md) for the fork, push, and pull request mechanics.

## Guardrails

- Keep migration compatibility changes separate from behavior changes where possible.
- Do not change copyright years.
- Do not change original authors.
- Do not invent migration script style without repo precedent.
- Do not run Odoo install/update/test commands until likely commands are discovered from repo docs, `$ODOO_TOOL_README`, or `$ODOO_BASE_COMMAND`, and confirmed by the user.
- Do not run `git filter-branch` or `git filter-repo` without explicit user approval, even when the module rename makes them the correct move.
- Do not add remotes, push branches, post migration-issue comments, or open pull requests without explicit user confirmation.
- Do not use `--no-verify` except for the OCA-documented first mechanical-formatting commit, and only after confirming that exception with the user.
- Prefer OCA PR title format: `[<target>][MIG] <module>: Migration to <target>`.
