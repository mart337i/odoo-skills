# Odoo Agent Setup

Use these skills when working on Odoo addons, OCA repositories, or module migrations.

## Environment

If the Odoo framework source or local development setup is not discoverable inside the workspace, ask the user to configure:

```bash
export ODOO_SOURCE=/path/to/odoo
export ODOO_BASE_COMMAND="/path/to/odoo/odoo-bin -c /path/to/odoo.conf --addons-path=/path/to/addons"
export ODOO_TOOL_README=/path/to/local-development/README.md
```

When `$ODOO_SOURCE` is set, inspect it for framework APIs, migration helpers, XML syntax, assets, tests, and version-specific behavior before guessing.

When `$ODOO_BASE_COMMAND` is set, treat it as the local starting point for Odoo commands. Inspect it before proposing install/update/test commands, but still ask the user to confirm before running anything that touches a database or local service.

When `$ODOO_TOOL_README` is set, read it before using local development tools, wrappers, docker compose files, invoke tasks, make targets, or repo-specific Odoo commands.

## Repo Detection

Treat a repo as Odoo-related when it contains signals such as:

- `__manifest__.py`
- `odoo-bin`
- `addons/`
- `models/`
- `views/`
- `security/ir.model.access.csv`
- `controllers/`
- `static/src/`
- Odoo branch names like `16.0`, `17.0`, `18.0`, or `19.0`

## Skill Routing

- Use `grill-me` when the user wants to stress-test a generic plan/design or says "grill me" outside an Odoo-specific context.
- Use `grill-with-docs` when the user wants to stress-test a plan against the project's existing domain language and documented decisions.
- Use `improve-codebase-architecture` when the user wants to find refactoring opportunities, consolidate tightly-coupled modules, improve testability, or make a codebase more AI-navigable.
- Use `zoom-out` when the user is unfamiliar with a section of code and needs broader context or a higher-level map.
- Use `odoo-grill-me` when the user wants to stress-test an Odoo plan/design, get grilled on addon architecture, clarify model/security/view choices before implementation, or says "grill me" in an Odoo context.
- Use `odoo-idea-gates` before building a new Odoo module, addon, marketplace app, or product idea when validating usefulness, existing alternatives, buyer demand, wedge, reach, Odoo fit, or whether the idea should exist at all.
- Use `odoo-owl` for OWL/frontend component work, Odoo web client components, `@odoo/owl`, templates, hooks, reactivity, props, registries, plugins, assets, frontend debugging, and OWL migrations.
- Use `odoo` for addon development, exploration, debugging, architecture, manifests, docs, security, views, models, controllers, assets, reports, and tests.
- Use `odoo-code-review` for Odoo code reviews, pull request reviews, correctness/security/performance audits, official Odoo coding guideline checks, migration risk reviews, and test coverage reviews.
- Use `odoo-code-tracer` when you need to trace execution from an entry point through controllers, buttons, cron jobs, model methods, overrides, computes, onchanges, constraints, database operations, side effects, and security checks.
- Use `odoo-migration` for OCA module migration, Odoo major-version ports, `[MIG]` PRs, and version checklist work.
- Use `odoo-test-writer` when creating or improving Odoo tests with TransactionCase, SingleTransactionCase, HttpCase, AccountTestInvoicingCommon, Form helper, tags, mocked external APIs, access tests, workflow tests, or test coverage improvements.
- Use `odoo-17.0`, `odoo-18.0`, or `odoo-19.0` as version-specific references when the target version is known and detailed guidance is needed for actions, controllers, data, decorators, fields, manifests, migrations, mixins, models, OWL, performance, reports, security, testing, transactions, translations, or views.
- Do not use OpenUpgrade workflows for `odoo-migration`; that skill is scoped to normal OCA module migration.

## Command Safety

Before running Odoo install/update/test commands, inspect repo docs/config, `$ODOO_TOOL_README`, and `$ODOO_BASE_COMMAND` for the intended command, then ask the user to confirm. Odoo commands often need a database, addons path, config file, and local service assumptions.
