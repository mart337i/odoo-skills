---
name: odoo-report-migration
description: Migrates Odoo QWeb HTML and PDF reports between major versions while preserving data semantics, access behavior, paper geometry, and intended visual appearance. Use when porting reports across Odoo versions, adapting report templates or assets after an upgrade, or diagnosing visual regressions in migrated PDFs.
---

# Odoo Report Migration

Use this skill for report-specific migration work. Compose it with
`odoo-migration` for the module port, the source and target version skills for
framework details, `odoo-report-design` for layout decisions, and
`odoo-web-testing` when browser or asset behavior is involved.

## First Move

Identify the source and target versions, module and report XML IDs, current
rendering environment, report action, template chain, paper format, assets,
custom value provider, and the visual requirements that must remain stable.

## OCA Process Still Applies

Report work is not process-free. When a report migration is part of a module
`[MIG]`, the full OCA process in
[odoo-migration](../odoo-migration/SKILL.md) still applies end to end:
announcing the module(s) on the `Migration to version <target>` issue,
creating the `<target>-mig-<module>` branch, the `[MIG]` commit, pushing to
the contributor's fork, and opening the pull request titled
`[<target>][MIG] <module>: Migration to <target>`. Rendering equivalence is
an acceptance criterion for the pull request, not a substitute for it. Do not
post, push, or open the pull request without the same explicit confirmation
required by `odoo-migration`.

## Workflow

1. Freeze a baseline. Render HTML and PDF using deterministic records and
   record the version, commit, language, company, user, paper format, and
   renderer details.
2. Inventory dependencies: inherited templates, external layouts, report
   assets, SCSS/CSS, fonts, images, paper formats, translations, and report
   actions.
3. Apply every intermediate Odoo migration checklist for a multi-version jump.
   Separate API/data migration from presentation changes.
4. Inspect the target version's upstream report templates, asset bundles, CSS
   framework, renderer, and report action fields before porting XPath or CSS.
5. Port the smallest compatible change. Prefer stable inherited anchors over
   copying the old core template.
6. Re-render the same fixture set and classify every difference as intended,
   renderer noise, or a regression.
7. Verify access, data semantics, translations, pagination, paper geometry,
   assets, and multi-record behavior before declaring success.

## Guardrails

- Do not call a report visually preserved because it merely loads or produces a PDF.
- Do not compare PDF bytes as the primary visual assertion.
- Do not fix a layout regression with arbitrary CSS before checking target
  templates, asset bundles, paper format, fonts, and renderer behavior.
- Do not broaden report access or expose data while changing presentation.
- If the baseline cannot be rendered, record the missing evidence and use a
  clearly labeled reconstructed baseline instead of claiming equivalence.

## References

- [REPORT-MIGRATION.md](REPORT-MIGRATION.md) - end-to-end migration procedure.
- [VISUAL-REGRESSION.md](VISUAL-REGRESSION.md) - practical output comparison workflow.
- [REPORT-VERSION-MATRIX.md](REPORT-VERSION-MATRIX.md) - version-sensitive audit points, 8.0 to 20.0.

The OCA version guides themselves are at
`https://github.com/OCA/maintainer-tools/wiki/Migration-to-version-<version>`.
The wiki is authoritative over this pack.
