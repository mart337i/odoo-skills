---
name: odoo-web-security
description: Security review and implementation guidance for Odoo website, portal, public controllers, JSON routes, QWeb output, and browser-facing assets. Use when exposing Odoo data over HTTP, adding portal/public pages or forms, using sudo, handling tokens, rendering HTML, or reviewing web attack surfaces.
---

# Odoo Web Security

Use this skill at every Odoo web trust boundary. Compose it with `odoo-website`, `odoo-code-review`, and the version-specific Odoo skill.

## Threat Model First

Identify the caller, authentication mode, record being accessed, mutation being performed, and data that crosses the browser boundary. Treat route parameters, query strings, form fields, JSON payloads, headers, cookies, and browser state as attacker-controlled.

Read [SECURITY-CHECKLIST.md](references/SECURITY-CHECKLIST.md) before editing or approving a web-facing change.

## Non-Negotiable Checks

- `auth='public'` is not a substitute for a publication policy.
- Portal identity and record ownership must be checked server-side.
- `sudo()` must be narrow, justified, and followed by an explicit visibility check.
- Never authorize a record from its numeric ID, token shape, or hidden form field alone.
- Use ORM security and record rules where possible; do not bypass them with raw SQL or broad elevation.
- Escape output by default and review HTML sanitization separately from escaping.
- Validate redirects, URLs, uploaded files, and external API destinations.
- Preserve CSRF protection for state-changing browser requests according to the route type and target version.

## Review Output

For each finding, state the attacker, entry point, missing control, affected data/action, and a concrete test that would fail before the fix.
