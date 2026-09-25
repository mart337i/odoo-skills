# Odoo Development

Use for creating or modifying Odoo addons.

## Workflow

1. Detect the addon/module and target Odoo version. If version is not discoverable, ask.
2. Read the relevant manifest, imports, models, views, security, data, controllers, tests, and docs before editing.
3. Search the installed Odoo addons and relevant OCA repositories for an existing model, controller, view, pattern, or module before inventing a parallel implementation.
4. Identify the smallest Odoo-native change: model extension, view inheritance, wizard, controller, report, data record, asset, or test.
5. Check [CONVENTIONS.md](CONVENTIONS.md) before writing.
6. Update manifest imports/data/dependencies only when the change requires it.
7. Add or adjust tests at the Odoo behavior seam when practical.
8. Inspect likely module update/test commands from repo docs/config, then ask the user to confirm before running them.

## Creating Addons

You may use available Odoo helper tooling to scaffold a module, but never trust generated output blindly. Verify naming, manifest dependencies, imports, access rights, data ordering, views, and repo conventions.

Minimum viable addon:

- `__init__.py`
- `__manifest__.py`
- relevant `models/` imports
- `security/ir.model.access.csv` for persistent models
- views/actions/menus only when user-facing UI is required
- tests when behavior is non-trivial or bug-prone

## Modifying Addons

- Extend existing models with `_inherit` when changing standard Odoo behavior.
- Create new models only when the domain concept is real and needs its own lifecycle/security.
- Prefer narrow inherited XML views over replacing base views.
- Keep business rules in models or service-like methods behind clear seams; avoid burying them in controllers, views, or onchange-only logic.
- Do not add dependencies for convenience. Add them only for actual imports, inherited models, XML refs, assets, external IDs, or data records.

## Frontend And Assets

For OWL, QWeb, website, portal, or asset work:

- Verify target Odoo version and consult docs or `$ODOO_SOURCE` for frontend API details.
- Inspect asset bundle definitions in the manifest and existing `static/src/` conventions.
- Keep JS components small and tied to clear UI behavior.
- Check template inheritance selectors and asset load order.
