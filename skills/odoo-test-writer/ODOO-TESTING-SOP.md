# Odoo Testing SOP

This SOP guides testing Odoo custom modules with the Odoo test framework. It covers simple unit-style tests without external dependencies and integration tests with mocked external API requests.

Version: 2.0  
Odoo focus: 18.0, with version-specific checks via local source or `odoo-17.0`, `odoo-18.0`, `odoo-19.0`, and `odoo-20.0` reference skills.

## Key Concepts

- `TransactionCase`: standard base class. Each test method runs inside its own savepoint and rolls back after completion.
- `SingleTransactionCase`: all test methods run in the same transaction. Faster for read-only suites, but state can leak between tests.
- `HttpCase`: extends `TransactionCase` with browser automation and URL helpers for end-to-end tests.
- `AccountTestInvoicingCommon`: accounting test base with preconfigured company, products, partners, taxes, and journals.
- Tagged tests: `@tagged` categorizes tests for selective execution.
- Mocking: replaces external dependencies such as APIs, gateways, OAuth, filesystem, time, or slow services with controlled fakes.
- `Form` helper: simulates user interactions with Odoo forms, including defaults, onchanges, validations, and x2many edits.

## Concepts Explained

- Transaction: a group of database operations that either succeed or fail together.
- Savepoint: a checkpoint inside a transaction. `TransactionCase` rolls back to a savepoint after each test.
- Mocking: replacing an external dependency with a controlled fake response so tests are fast and deterministic.
- Test fixture: reusable records/helpers prepared for tests, often in `setUpClass`.
- Isolation: each test must run independently and not depend on side effects from another test.
- Assertion: a check that fails the test if the expected condition is not true.

## Directory Structure

```text
your_module/
├── __init__.py              # Do not import tests here
├── __manifest__.py
├── models/
├── tests/
│   ├── __init__.py          # Import test modules here
│   ├── common.py            # Shared fixtures/helpers
│   └── test_*.py            # Test files
└── views/
```

`tests/__init__.py` must import test modules:

```python
from . import test_your_feature
```

Important: `your_module/__init__.py` must not import `tests`. Test modules belong only in `tests/__init__.py`.

## Test Class Selection

| Class | Transaction Behavior | Use Case |
|---|---|---|
| `TransactionCase` | Each test in savepoint | Standard model, workflow, compute, validation, and access tests |
| `AccountTestInvoicingCommon` | TransactionCase-style | Accounting/invoicing tests needing account fixtures |
| `SingleTransactionCase` | One transaction for all tests | Read-only tests or expensive setup where shared state is safe |
| `HttpCase` | TransactionCase + browser/HTTP helpers | Website, portal, tours, browser JS, and controller integration |

## Test Organization

- One test class per feature/model/workflow.
- Use descriptive method names such as `test_01_description_of_behavior`.
- Use numbering only when order helps readability; do not rely on order for isolation.
- Use docstrings for non-obvious scenarios.
- Follow Arrange, Act, Assert.
- Create immutable shared fixtures in `setUpClass`.
- Create mutable records inside each test method.
- Prefer helper methods in `common.py` for frequently-created records.

## Tags

Common tags:

- `standard`: default tag for tests inheriting from Odoo base cases.
- `at_install`: runs right after module installation; default for many tests.
- `post_install`: runs after all modules are installed.
- `-at_install`: explicitly excludes the test from install-time execution.
- `-standard`: removes from default test suite; useful for slow or optional tests.

Typical post-install test class:

```python
from odoo.tests import tagged

@tagged("post_install", "-at_install")
class TestYourFeature(YourModuleTestCommon):
    pass
```

Tag selector syntax:

```text
[-][tag][/module][:class][.method]
```

## Common Imports

```python
from datetime import date, datetime, timedelta
from unittest.mock import MagicMock, call, patch

from odoo.exceptions import AccessError, UserError, ValidationError
from odoo.tests import HttpCase, SingleTransactionCase, TransactionCase, tagged
from odoo.tests.common import Form
from odoo.addons.account.tests.common import AccountTestInvoicingCommon
```

## Mocking Guidelines

- Mock external dependencies, not normal Odoo model behavior.
- Patch where the object is used, not where it originally comes from.
- For `import requests`, patch `requests.post` if the code calls `requests.post`.
- For `from requests import post`, patch `your_module.models.your_model.post`.
- Assert both Odoo state changes and mock calls.
- Test success, timeout, server error, authentication failure, invalid payload, and retry/polling behavior when relevant.

## Form Helper Guidance

Use `Form` when regular `create()`/`write()` would bypass form-only behavior:

- default values
- `@api.onchange`
- One2many line editing
- Many2many tags
- form validations
- user-like workflows

Do not rely on onchange-only logic for persisted business rules. If persisted behavior matters, test the model method/constraint too.

## Access And Security Tests

For security-sensitive changes, include tests with realistic users:

- `with_user(test_user)` for user-specific access.
- groups assigned via XML refs.
- company-specific data and record rules.
- portal/public ownership checks for controllers.
- `AccessError` for forbidden operations.

Review groups, ACLs, record rules, domains, `sudo()`, and multi-company isolation together.

## Performance Tests

Use query-count checks for performance-sensitive code when available:

```python
with self.assertQueryCount(__limit__=5):
    records.action_compute_batch()
```

Target N+1 risks:

- `search()` in loops
- `browse()` in loops
- per-record `create()`, `write()`, or `unlink()`
- computed fields that trigger excessive related record loads

## Troubleshooting

Tests pass alone but fail together:

- shared mutable fixture pollution
- reliance on test order
- missing rollback isolation because of `SingleTransactionCase`

Mock not applied:

- wrong patch path
- module imported the function directly
- code uses a session/client object instead of `requests.*`

Database constraint errors:

- missing required related records
- missing company/currency/journal/account setup
- wrong accounting test base

Access errors:

- user lacks required groups
- record rule domain excludes the record
- company context is wrong
- code accidentally relies on admin permissions

Timeouts:

- unmocked external call
- polling loop with no terminal response
- sleep not patched

## Testing Pyramid

```text
           /\
          /  \
         / UI \        Few: HttpCase, tours, browser flows
        /------\
       /        \
      /Integration\   Some: Form tests, multi-model workflows
     /------------\
    /              \
   /  Unit Tests    \ Many: TransactionCase model behavior
  /------------------\
```

Aim for roughly:

- 80% `TransactionCase` model/business tests.
- 15% integration tests with `Form` or multi-model workflows.
- 5% `HttpCase`/tour tests for critical user journeys.

## Completion Checklist

- `tests/__init__.py` imports new test modules.
- Module root `__init__.py` does not import tests.
- Tests use the smallest suitable Odoo base class.
- External dependencies are mocked at the boundary.
- Access tests use realistic users/groups/companies when relevant.
- Tests cover success and failure paths.
