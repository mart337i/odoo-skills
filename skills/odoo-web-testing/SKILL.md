---
name: odoo-web-testing
description: Odoo browser-facing test guidance for HTTP controllers, website and portal flows, QWeb templates, assets, OWL integration, browser tours, and JavaScript behavior. Use when testing or debugging public pages, portal access, forms, controllers, frontend assets, or end-to-end Odoo web behavior.
---

# Odoo Web Testing

Use this skill to test behavior at the browser-facing seam rather than only testing isolated controller methods. Compose it with `odoo-test-writer`, `odoo-website`, and `odoo-web-security`.

## First Move

Identify the user journey and its boundary:

- HTTP response, redirect, or rendered template.
- ORM mutation or access rule.
- Asset bundle and template loading.
- Browser interaction, form submission, or asynchronous RPC.
- User roles and companies that must be isolated.

Read [BROWSER-TESTING.md](references/BROWSER-TESTING.md) for HTTP/browser flows
and [OWL-TESTING.md](references/OWL-TESTING.md) for components, services,
registries, and field widgets.

## Test Strategy

- Use a model test for business rules that do not require HTTP.
- Use an HTTP test for route status, auth, redirects, ownership, CSRF, and rendered content.
- Use a browser/tour test for real interaction, assets, templates, client state, and navigation.
- Use JavaScript unit tests for isolated client behavior and service/registry integration.
- Add negative tests for anonymous access, another portal user's record, invalid IDs, invalid tokens, and cross-company data.

## Always Apply

- Assert externally visible behavior, not incidental HTML formatting.
- Keep fixtures small and create users with the exact groups needed.
- Make asynchronous behavior deterministic; do not add arbitrary sleeps.
- Test both success and failure paths.
- State when browser, database, or asset-bundle verification was not run.
