---
name: odoo
description: Odoo engineering workflows for addon development, codebase exploration, debugging, architecture/refactor review, manifest/docs sync, and routing to migration work. Use when the user mentions Odoo, addons, modules, manifests, models, XML views, security CSVs, record rules, Odoo shell, OWL/QWeb/assets, or when an Odoo codebase is detected.
---

# Odoo

Use this skill only for Odoo work. For OCA module migration or major-version addon ports, route to `odoo-migration`.

## First Move

Explore before asking. Inspect the repo for Odoo signals: `__manifest__.py`, `odoo-bin`, `addons/`, `models/`, `views/`, `security/ir.model.access.csv`, `controllers/`, `static/src/`, and project docs.

Before version-sensitive changes, detect the target Odoo version from branch names, manifests, docs, dependencies, or config. If the version is not discoverable, ask. If `$ODOO_SOURCE` is set, inspect it for framework behavior instead of guessing.

Before running Odoo update/test commands, inspect likely commands from repo docs/config. If `$ODOO_TOOL_README` is set, read it for local tooling. If `$ODOO_BASE_COMMAND` is set, use it as the starting point for proposed Odoo commands. Then ask the user to confirm before running commands that touch a database or local service.

## Route The Task

- Stress-test an Odoo plan/design before implementation: use `odoo-grill-me`.
- Build or change addon behavior: use [DEVELOPMENT.md](DEVELOPMENT.md).
- Understand an unfamiliar Odoo codebase: use [EXPLORATION.md](EXPLORATION.md).
- Debug broken Odoo behavior: use [DEBUGGING.md](DEBUGGING.md).
- Trace a concrete execution path through controllers, buttons, cron jobs, model methods, overrides, computes, onchanges, constraints, or side effects: use `odoo-code-tracer`.
- Hunt for Odoo architecture/refactor opportunities: use [ARCHITECTURE.md](ARCHITECTURE.md).
- Sync manifest/docs with implemented behavior: use [MANIFEST-DOCS.md](MANIFEST-DOCS.md).
- Build, review, debug, or migrate Odoo OWL frontend components: use `odoo-owl`.
- Review Odoo code for correctness, security, performance, tests, migrations, manifests, and official coding guidelines: use `odoo-code-review`.
- Create or improve Odoo tests using TransactionCase, HttpCase, Form helper, tags, mocks, access tests, workflow tests, or test coverage patterns: use `odoo-test-writer`.
- Migrate an OCA addon between Odoo major versions: use `odoo-migration`.
- Need detailed known-version guidance: use `odoo-17.0`, `odoo-18.0`, or `odoo-19.0`.

## Always Apply

Use the compact conventions in [CONVENTIONS.md](CONVENTIONS.md). Keep changes small and idiomatic. Include a security pass when models, views, controllers, workflows, or access behavior change. For business data modules, lightly check multi-company behavior: `company_id`, defaults, domains, record rules, and cross-company reads/writes.
