# OWL 3 And Odoo Integration

Use this when OWL code lives inside an Odoo addon. Verify local Odoo/OWL version before applying OWL 3 patterns.

## Version Detection

Before editing Odoo frontend code, inspect:

- Odoo branch/version (`16.0`, `17.0`, `18.0`, `19.0`, master, etc.).
- `$ODOO_SOURCE` when set.
- Local OWL source under `addons/web/static/lib/owl` or equivalent.
- Existing imports in the addon and nearby web modules.
- Existing frontend tests and asset bundle conventions.

Do not mix OWL 3 APIs into an OWL 1/2 Odoo codebase unless the task is explicitly a migration and the compatibility path is clear.

## Asset Integration

Inspect `__manifest__.py` assets before adding files. Odoo modules commonly place frontend code under:

- `static/src/js/`
- `static/src/xml/`
- `static/src/scss/`
- `static/tests/`

Check whether the target version expects XML templates in a specific bundle and whether JS module headers/import paths follow local conventions.

When adding templates, ensure the XML file is listed in the correct asset bundle and loaded before use.

## Template Strategy

Prefer external XML templates in Odoo addons when:

- user-facing strings need translation extraction
- templates are large or shared
- Odoo asset tooling expects XML files
- inheritance/overrides are part of the design

Inline `xml` templates are fine for small local components, tests, examples, and standalone OWL apps.

## Odoo Web Client Conventions

Inspect the target Odoo source for the current patterns before choosing APIs. Depending on version, Odoo may use framework-specific services, registries, hooks, action managers, view registries, field widgets, or compatibility layers.

When integrating with Odoo web client:

- Register components/widgets/actions in the appropriate Odoo registry for that version.
- Use Odoo-provided services/hooks when the local version exposes them.
- Keep data access behind services/models instead of embedding RPC details in deep UI components.
- Keep component props narrow and explicit.
- Avoid direct DOM manipulation except at integration boundaries with external libraries.

## Styling And Guidelines

Compose with `odoo-code-review` for file naming, static organization, JavaScript, and SCSS conventions.

Use Odoo SCSS class conventions:

- `o_<module_or_component>` prefix.
- Avoid `id` selectors.
- Avoid hyper-specific nested names.
- Keep component styles colocated in `static/src/scss` where repo convention expects them.

## Testing And Debugging

Before proposing commands, inspect `$ODOO_TOOL_README`, `$ODOO_BASE_COMMAND`, repo docs, and package scripts if present.

Useful seams:

- frontend unit tests if the repo has them
- web tour/browser tests
- Odoo module update to verify asset loading
- browser console and network tab
- OWL DevTools where available
- template load/translation extraction checks

Ask before running commands that touch an Odoo database or local service.

## Common Odoo Frontend Failure Modes

- Asset file exists but is not in the correct manifest bundle.
- Template name mismatch or template not loaded before component use.
- Import path follows another Odoo version's convention.
- Component registered in the wrong registry/category.
- Props mutated by child instead of sending event/callback to parent.
- Event handler loses `this` because callback was not bound or `.bind` suffix was not used where applicable.
- External listener or interval is not cleaned up.
- Async response writes to a destroyed component.
- Unsafe HTML output bypasses escaping without a reviewed trust boundary.
