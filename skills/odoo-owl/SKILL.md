---
name: odoo-owl
description: Odoo OWL frontend engineering guidance covering components, templates, reactivity, hooks, props, plugins, registries, debugging, assets, and OWL 1/2/3 migration. Use when working with Odoo frontend components, @odoo/owl, static/src JavaScript/XML templates, signals, proxy, props, t-on, t-ref, t-model, registries, plugins, or OWL migrations.
---

# Odoo OWL

Use this skill for OWL frontend work in Odoo addons or standalone Owl applications.

## First Move

Do not assume OWL 3. First detect the actual OWL/Odoo frontend version from local source and project code:

- Odoo version from branch, manifests, docs, or config.
- Local OWL source under `$ODOO_SOURCE` when set, especially `addons/web/static/lib/owl` or equivalent.
- Existing imports from `@odoo/owl`, `owl`, `@web/*`, and local frontend conventions.
- Asset declarations in `__manifest__.py` and existing `static/src/js`, `static/src/xml`, and `static/src/scss` patterns.

Use the OWL 3 references only when the target project actually uses OWL 3. For older Odoo versions, local source and existing project conventions supersede this skill's OWL 3 notes.

## Route The Task

- Build or change OWL components: use [OWL3-CORE.md](OWL3-CORE.md), [OWL3-TEMPLATES.md](OWL3-TEMPLATES.md), and [OWL3-STATE-LIFECYCLE.md](OWL3-STATE-LIFECYCLE.md).
- Integrate with Odoo assets/web client: use [OWL3-ODOO-INTEGRATION.md](OWL3-ODOO-INTEGRATION.md).
- Migrate OWL 1/2 code toward OWL 3: use [OWL3-MIGRATION.md](OWL3-MIGRATION.md), but verify local compatibility first.
- Review frontend guideline/style concerns: compose with `odoo-code-review`.
- Debug broken Odoo frontend behavior: compose with `odoo` debugging workflow and inspect browser/asset/test seams.

## Always Apply

Keep UI state explicit and reactive. Prefer small components with clear props and events. Avoid mutating props. Clean up listeners and external resources with lifecycle hooks. Treat unsafe HTML as a security boundary. Verify asset bundle registration and template loading before blaming OWL behavior.
