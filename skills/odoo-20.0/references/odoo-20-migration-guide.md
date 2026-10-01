# Odoo 20 Migration Guide

Guide to version-sensitive core patterns to review when migrating modules to
Odoo 20. Use [`odoo-migration`](../../odoo-migration/SKILL.md) for the
repository-level OCA migration workflow.

## Table of Contents
- [Migration Overview](#migration-overview)
- [Key Changes](#key-changes)
- [Migration Scripts](#migration-scripts)
- [Module Hooks](#module-hooks)
- [Common Migrations](#common-migrations)
- [Testing](#testing)

---

## Migration Overview

### When to Migrate

- **Major version upgrade**: Odoo 17 → 18 → 19 → 20
- **Module dependencies changed**
- **API breaking changes**

### Migration Strategy

1. Review changelog and breaking changes
2. Update `__manifest__.py` version
3. Run migration scripts
4. Test thoroughly
5. Update documentation

---

## Key Changes

### Odoo 20 Patterns To Verify

| Area | Pattern to verify in the target source |
|------|------------------------------------------|
| Views | Use the target version's supported list and conditional-attribute syntax |
| Delete validation | Prefer the target version's documented `@api.ondelete` pattern where applicable |
| Aggregation | Use the target version's documented field aggregation parameter |
| SQL queries | Prefer ORM utilities; use `odoo.tools.SQL` and correct flush/invalidation when SQL is necessary |
| Domains | Use `odoo.fields.Domain` when composing user- or code-provided domains |
| Constraints and indexes | Review `models.Constraint`, `models.Index`, and `models.UniqueIndex` declarations |

### Odoo 20 Release-Note Watchlist

These are development-relevant release-note items to check during 20.0 work. Verify exact field names in the target source before editing production modules.

| Area | Migration impact |
| ---- | ---------------- |
| Runtime | Use Python 3.12+ and PostgreSQL 16+ in dev containers, CI, test databases, and deployments. |
| Security | Access rows are `ir.access` records with `operation` and `domain`; older `ir.model.access`/`perm_*` and `ir.rule` examples are stale for new 20.0 guidance. |
| Access APIs | Replace old access checks with `check_access`, `has_access`, `_filtered_access`, `_access_domain`, `has_field_access`, and `check_field_access`. |
| Groups | `res.groups.category_id` moved to `res.groups.privilege_id`; the category lives on `res.groups.privilege`. |
| Technical group fields | Use `group_ids` on views, menus, actions, reports, users, and related technical records where the target source expects it. |
| ORM grouping | `read_group` uses the final backend signature and tuple-style results; use `formatted_read_group` for old web-style dictionaries. |
| Domains | Imports from `odoo.osv.expression` are removed; migrate to `odoo.fields.Domain` or `odoo.orm.domains.Domain`. |
| Reports | PDF rendering is engine-based; use engine-specific report types and `_run_pdf_engine*` overrides. |
| Tests | Untagged tests default to `standard, post_install`; `SingleTransactionCase` is removed. |
| XML/data | Use `type="bytes" file="..."`, remove legacy manifest keys, and review XML function calls for unsafe names. |
| OWL/frontend | OWL 3 uses `useProps`, plugin/resource APIs, Hoot tests, and website `Interaction` classes instead of old public widgets. |
| Mail tracking | Stored tracking values were removed from the default tracking framework; tracking messages are generated on the fly. Audit code that queries `mail.tracking.value`. |
| Outgoing mail | `mail.mail` keeps source content in `body_html`; `body_content` is computed from it. |
| Many2one fields | Backend synchronization and large-list behavior changed; test custom widgets/domains after onchange/save. |
| Icons | Font Awesome was replaced by Material Symbols in the web client; inspect target templates before copying `fa-*` classes. |
| Push notifications | Firebase is no longer the default push path; use Odoo internal push tooling where applicable. |
| Offline mode | Standard web flows can create, edit, archive, unarchive, delete, and repeat previous searches offline; custom web code should account for retry/conflict behavior. |
| Translations | Translation mode, multilingual import/export, and a text/HTML translation wizard affect data-import and report-review workflows. |
| Website/portal | Review translated/static URLs, Unicode normalization, website form limits, SEO defaults, portal card customization, and page-specific body classes. |

### App And Model Changes To Audit

- Accounting: parent accounts replace account groups; asset models are replaced by depreciation models; payment statuses were renamed; payment reconciliation and bank-origin constraints changed; taxes are constrained by fiscal positions.
- Inventory/MRP/POS: review return wizard removal, retroactive COGS valuation behavior, valuation without Inventory, removed flexible consumption setting, HTML manufacturing notes, and POS end-session accounting changes.
- Sales/eCommerce/Rental/Subscriptions: sale order lines no longer require products by default; Sales, Subscription, and Rental use a unified pricelist engine; Mondial Relay eCommerce is replaced by Sendcloud; guest-order contacts are archived by default.
- HR/Planning/Payroll: Field Service moved into Planning; work entries and some Planning/Payroll integration flows were removed; salary rules can belong to multiple categories/structures.
- Payments: SOFORT is removed; review provider tokenization, webhook, OAuth, PayPal/Auth.net, and provider restriction changes before touching payment modules.

---

## Migration Scripts

### Migration Script Location

```
my_module/
└── migrations/
    └── 20.0.1.0/
        ├── pre-migration.py
        ├── end-migration.py
        └── post-migration.py
```

### Migration Script Naming

| Script | When it runs |
|--------|--------------|
| `pre-migration.py` | Before module update |
| `post-migration.py` | After module update |
| `end-migration.py` | After all migrations |

### Migration Script Template

```python
def migrate(cr, version):
    """
    Migration script for Odoo 20
    """
    # Your migration code here
    pass
```

---

## Common Migrations

### Tree to List View

**Older versions**:

```xml
<tree string="Records">
    <field name="name"/>
</tree>
```

**Versions supporting list views**:

```xml
<list string="Records">
    <field name="name"/>
</list>
```

### attrs to Direct Attributes

**Older versions**:

```xml
<field name="state" attrs="{'invisible': [('state', '!=', 'draft')]}"/>
```

**Versions supporting direct attributes**:

```xml
<field name="state" invisible="state != 'draft'"/>
```

### Delete Validation

**Older versions**:

```python
def unlink(self):
    for record in self:
        if record.state != 'draft':
            raise UserError("Cannot delete non-draft records")
    return super().unlink()
```

**Versions supporting `@api.ondelete`**:

```python
@api.ondelete(at_uninstall=False)
def _unlink_if_not_draft(self):
    if any(rec.state != 'draft' for rec in self):
        raise UserError("Cannot delete non-draft records")
```

### Field Aggregation

**Older versions**:

```python
amount_total = fields.Monetary(group_operator="sum")
```

**Versions supporting `aggregator=`**:

```python
amount_total = fields.Monetary(aggregator="sum")
```

Odoo 20 no longer translates deprecated `group_operator=` to `aggregator=`. Search and replace it during migration instead of relying on warnings.

---

## Module Hooks

### pre_init_hook

```python
def pre_init_hook(env):
    """Called before module installation"""
    # Create custom tables, etc.
    pass
```

### post_init_hook

```python
def post_init_hook(env):
    """Called after module installation"""
    # Set default values, create records, etc.
    pass
```

### uninstall_hook

```python
def uninstall_hook(env):
    """Called after module uninstallation"""
    # Clean up custom tables, files, etc.
    pass
```

### Register Hooks in Manifest

```python
{
    ...
    'pre_init_hook': 'my_module.pre_init_hook',
    'post_init_hook': 'my_module.post_init_hook',
    'uninstall_hook': 'my_module.uninstall_hook',
}
```

---

## Migration Checklist

### Highest-Risk First Pass

1. Migrate security files and record rules to `ir.access`.
2. Scan Python for old access APIs, old `read_group`, and `odoo.osv.expression`.
3. Scan XML/QWeb for `t-esc`, `t-raw`, old manifest keys, report overrides, and old binary data loading.
4. Scan frontend code for OWL 2 props/effects/env usage and legacy public widgets.
5. Migrate JS tests from QUnit/env helpers to Hoot/app helpers.
6. Update CI/dev environment to Python 3.12+ and PostgreSQL 16+.

### Framework Search Patterns

```text
security/ir.model.access.csv
<record model="ir.rule"
check_access_rights
check_access_rule
_filter_access_rules
_filter_access_rules_python
check_field_access_rights
_check_access
read_group(
odoo.osv.expression
t-esc
t-raw
init_xml
update_xml
demo_xml
api.deprecated
SingleTransactionCase
auth="bearer"
type="base64" file=
group_operator=
```

### Frontend Search Patterns

```text
static props
defaultProps
useEffect(
env.debug
env.isSmall
Component.env
@web/legacy/js/public/public_widget
root.widget
@web/legacy/js/public/minimal_dom
_service.js
env_test_helpers
QUnit.
web.qunit_suite_tests
/web/tests/legacy
```

### Migration Direction

- Convert ACLs and record rules into `ir.access` rows with `operation` and `domain`.
- Use `base.group_everyone` for all-user grants; keep empty `group_id/id` only for global restrictions.
- Replace old access checks with `check_access`, `has_access`, `_filtered_access`, and `_access_domain`.
- Replace old `read_group` signatures with Odoo 20 aggregate/groupby signatures or `formatted_read_group`.
- Replace `odoo.osv.expression` helpers with `fields.Domain` or `orm.domains.Domain`.
- Replace `t-esc` and `t-raw` with `t-out` and explicit safe markup where needed.
- Move legacy manifest data keys into `data` or `demo`.
- Replace `group_operator` with `aggregator`.
- Update bearer routes to provide `bearer_scope`.
- Replace XML binary file data with `type="bytes"`.
- Convert OWL 2 props to `useProps(...)` and migrate OWL 2 effects.
- Replace public widgets with `Interaction` classes registered in `public.interactions`.
- Prefer plugin-aware APIs for converted frontend services.
- Migrate JS tests from QUnit-era helpers to Hoot/app helpers.

---

## Testing Migrations

### Test Migration Script

```python
from odoo.tests import TransactionCase

class TestMigration(TransactionCase):
    def test_migration(self):
        # Test migration script
        pass
```

### Manual Testing

1. Install previous version with sample data
2. Update to Odoo 20
3. Verify all data migrated correctly
4. Test all features

---

## References

- Odoo 19 changelog
- Odoo 20 official documentation and source
