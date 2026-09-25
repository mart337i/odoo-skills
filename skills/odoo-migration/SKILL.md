---
name: odoo-migration
description: End-to-end OCA module migration workflow for porting addons between Odoo major versions and preparing [MIG] pull requests. Use when migrating OCA modules, porting addons between Odoo versions, applying OCA migration checklists, resolving migration patch conflicts, or preparing versioned OCA migration PRs.
---

# Odoo Migration

Use this skill for OCA module migration between Odoo major versions. Compose
with `odoo-report-migration` when the module contains QWeb/PDF reports whose
visual output must be preserved.

This is not an OpenUpgrade skill. Do not use OpenUpgrade workflows, commands, `upgrade_analysis.txt`, or data-model migration analysis as the operating model. If a migration requires project-specific data handling, inspect the addon and repo conventions directly.

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

## Guardrails

- Keep migration compatibility changes separate from behavior changes where possible.
- Do not change copyright years.
- Do not change original authors.
- Do not invent migration script style without repo precedent.
- Do not run Odoo install/update/test commands until likely commands are discovered from repo docs, `$ODOO_TOOL_README`, or `$ODOO_BASE_COMMAND`, and confirmed by the user.
- Prefer OCA PR title format: `[<target>][MIG] <module>: Migration to <target>`.
