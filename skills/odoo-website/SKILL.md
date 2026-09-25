---
name: odoo-website
description: Odoo website and portal engineering guidance for QWeb templates, public and portal controllers, website models, frontend assets, and route integration. Use when working on Odoo website pages, portal flows, public forms, QWeb, website templates, portal access, or frontend behavior outside a pure OWL component.
---

# Odoo Website

Use this skill for Odoo website, portal, and public-facing addon work. Compose it with `odoo-owl` for reactive components, `odoo-web-security` for trust-boundary review, and the matching version skill for version-sensitive APIs.

## First Move

Identify the Odoo version and inspect:

- `website`, `portal`, and other relevant manifest dependencies.
- Existing controllers, routes, templates, and portal mixins.
- `auth`, route type, CSRF behavior, and ownership checks on nearby routes.
- Website asset bundles and template inheritance anchors.
- Existing tests for anonymous, portal, and internal users.

If `$ODOO_SOURCE` is set, verify controller, QWeb, asset, and portal behavior against the local source. Otherwise use the official documentation for the detected version and state any uncertainty.

## Route The Task

- Public or portal page flow: read [WEBSITE-PATTERNS.md](references/WEBSITE-PATTERNS.md).
- QWeb or template inheritance: read [QWEB-PATTERNS.md](references/QWEB-PATTERNS.md).
- Access, identity, or unsafe input: compose with `odoo-web-security`.
- OWL component or web-client registry work: compose with `odoo-owl`.
- Browser or controller verification: compose with `odoo-web-testing`.

## Always Apply

- Reuse the closest standard Odoo controller, template, mixin, and route convention.
- Treat every public or portal parameter as untrusted; validate record ownership before rendering or mutating data.
- Keep `sudo()` at explicit, narrow trust boundaries and never use it as an access check.
- Prefer `t-out` and safe structured values; treat raw HTML and `Markup` as security-sensitive.
- Use stable template anchors and preserve translations, responsive behavior, and asset ordering.
- Test the route as each relevant user type, including unauthenticated users when the route is public.
