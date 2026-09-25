---
name: odoo-test-writer
description: Create and improve Odoo custom module tests using the Odoo test framework. Use when the user asks to add tests, create TransactionCase/HttpCase tests, test Odoo business logic, mock external APIs, use Form helper, test access rules, test workflows, or improve Odoo test coverage.
---

# Odoo Test Writer

Use this skill to add practical tests for Odoo custom modules. Prefer tests at the Odoo behavior seam: ORM flows, constraints, computes, access rules, controllers, reports, forms, workflows, and integration boundaries.

## First Move

Before writing tests:

- Identify the addon/module and target Odoo version.
- Inspect `__manifest__.py`, `models/`, `views/`, `security/`, `controllers/`, `report/`, `static/src/`, existing `tests/`, and relevant docs.
- If `$ODOO_SOURCE` is set, inspect local test helpers and framework behavior instead of guessing.

## Choose The Test Type

- Use `TransactionCase` for most model/business-logic tests. Each test runs in a savepoint and rolls back.
- Use `AccountTestInvoicingCommon` for accounting tests needing companies, products, partners, taxes, journals, or invoices.
- Use `Form` helper when testing default values, onchanges, form validations, One2many/Many2many form behavior, or user-like form flows.
- Use `HttpCase` for browser, tour, website, portal, controller, or full UI flows.
- Use `SingleTransactionCase` only for read-only suites or expensive setup where cross-test state is acceptable.
- Use `unittest.mock.patch`/`MagicMock` for external APIs, payment gateways, OAuth, webhooks, file system, time, or slow services. Do not mock Odoo internals unless there is no better seam.

## Required Structure

```text
your_module/
├── __init__.py              # Do not import tests here
├── __manifest__.py
├── tests/
│   ├── __init__.py          # Import test modules here
│   ├── common.py            # Optional shared fixtures/helpers
│   └── test_*.py            # Test files
```

Every new `test_*.py` must be imported from `tests/__init__.py`, or the Odoo test runner will not discover it.

## Writing Workflow

1. Map the behavior to prove and the smallest Odoo seam that proves it.
2. Add or update `tests/common.py` only for reusable fixtures/helpers.
3. Create fresh mutable records inside each test unless immutable setup belongs in `setUpClass`.
4. Use Arrange, Act, Assert sections in test method body.
5. Test happy path, validation/error path, and relevant side effects.
6. For access/security changes, test realistic users/groups/companies with `with_user()`.
7. For external integrations, patch the boundary where it is used, then assert both state changes and mock calls.
8. For performance-sensitive code, add `assertQueryCount` when supported by the target version/test base.
9. Update `tests/__init__.py` and manifest dependencies only when evidence requires it.
10. For OWL components, services, registries, and field widgets, compose with `odoo-web-testing` and its `OWL-TESTING.md` reference.

## Tags

Default pattern for post-install module tests:

```python
from odoo.tests import tagged

@tagged("post_install", "-at_install")
class TestYourFeature(YourModuleTestCommon):
    pass
```

## References

- See [ODOO-TESTING-SOP.md](ODOO-TESTING-SOP.md) for the SOP, terminology, tags, and troubleshooting.
- See [ODOO-TESTING-EXAMPLES.md](ODOO-TESTING-EXAMPLES.md) for ready-to-adapt test patterns.
- Compose with `odoo-code-tracer` when you need to trace the behavior before writing tests.
- Compose with `odoo-code-review` when reviewing test coverage or test quality.
