---
name: odoo-19.0
description: Odoo 19 development knowledge base covering actions, controllers, data files, API decorators, SQL constraints, indexes, modules, fields, manifests, mixins, ORM methods, migrations, OWL components, performance, QWeb reports, security, testing, transactions, translations, and XML views. Use when writing, reviewing, or debugging Odoo 19 Python, XML, or JavaScript code, creating or modifying modules, fixing performance issues, or looking up Odoo 19 API patterns and best practices.
---

# Odoo 19 Skill - Master Index

Master index for all Odoo 19 development guides. Read the appropriate guide from `references/` based on your task.

## Quick Reference

| Topic          | File                                      | When to Use                                             |
| -------------- | ----------------------------------------- | ------------------------------------------------------- |
| Actions        | `references/odoo-19-actions-guide.md`     | Creating actions, menus, scheduled jobs, server actions |
| API Decorators | `references/odoo-19-decorator-guide.md`   | Using @api decorators, compute fields, validation       |
| Controllers    | `references/odoo-19-controller-guide.md`  | Writing HTTP endpoints, routes, web controllers         |
| Data Files     | `references/odoo-19-data-guide.md`        | XML/CSV data files, records, shortcuts                  |
| Development    | `references/odoo-19-development-guide.md` | Creating modules, manifest, reports, security, wizards  |
| Field Types    | `references/odoo-19-field-guide.md`       | Defining model fields, choosing field types             |
| Manifest       | `references/odoo-19-manifest-guide.md`    | **manifest**.py configuration, dependencies, hooks      |
| Migration      | `references/odoo-19-migration-guide.md`   | Upgrading modules, data migration, version changes      |
| Mixins         | `references/odoo-19-mixins-guide.md`      | mail.thread, activities, email aliases, tracking        |
| Model Methods  | `references/odoo-19-model-guide.md`       | Writing ORM queries, CRUD operations, domain filters    |
| OWL Components | `references/odoo-19-owl-guide.md`         | Building OWL UI components, hooks, services             |
| Performance    | `references/odoo-19-performance-guide.md` | Optimizing queries, fixing slow code, preventing N+1    |
| Reports        | `references/odoo-19-reports-guide.md`     | QWeb reports, PDF/HTML, templates, paper formats        |
| Security       | `references/odoo-19-security-guide.md`    | Access rights, record rules, field permissions          |
| Testing        | `references/odoo-19-testing-guide.md`     | Writing tests, mocking, assertions, browser testing     |
| Transactions   | `references/odoo-19-transaction-guide.md` | Handling database errors, savepoints, UniqueViolation   |
| Translation    | `references/odoo-19-translation-guide.md` | Adding translations, localization, i18n                 |
| Views & XML    | `references/odoo-19-view-guide.md`        | Writing XML views, actions, menus, QWeb templates       |

## File Structure

```
skills/odoo-19.0/
├── SKILL.md                          # This file - master index
└── references/                       # Development guides
    ├── odoo-19-actions-guide.md
    ├── odoo-19-controller-guide.md
    ├── odoo-19-data-guide.md
    ├── odoo-19-decorator-guide.md
    ├── odoo-19-development-guide.md
    ├── odoo-19-field-guide.md
    ├── odoo-19-manifest-guide.md
    ├── odoo-19-migration-guide.md
    ├── odoo-19-mixins-guide.md
    ├── odoo-19-model-guide.md
    ├── odoo-19-owl-guide.md
    ├── odoo-19-performance-guide.md
    ├── odoo-19-reports-guide.md
    ├── odoo-19-security-guide.md
    ├── odoo-19-testing-guide.md
    ├── odoo-19-transaction-guide.md
    ├── odoo-19-translation-guide.md
    └── odoo-19-view-guide.md
```

## Base Code Reference (Odoo 19)

All guides are based on analysis of Odoo 19 source code:

- `odoo/models.py` - ORM implementation
- `odoo/fields.py` - Field types
- `odoo/api.py` - Decorators
- `odoo/http.py` - HTTP layer
- `odoo/exceptions.py` - Exception types
- `odoo/tools/translate.py` - Translation system
- `odoo/addons/base/models/res_lang.py` - Language model
- `addons/web/static/src/core/l10n/translation.js` - JS translations

## External Documentation

- [Odoo 19 Official Documentation](https://github.com/odoo/documentation/tree/19.0)
- [Odoo 19 Developer Reference](https://github.com/odoo/documentation/blob/19.0/developer/reference/orm.rst)
