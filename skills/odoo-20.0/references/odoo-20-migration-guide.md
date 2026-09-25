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

- [ ] Review Odoo 19 to Odoo 20 upstream changes and module dependencies
- [ ] Update `__manifest__.py` version
- [ ] Update dependencies
- [ ] Verify view syntax against Odoo 20 source
- [ ] Review delete validation and field aggregation patterns
- [ ] Review SQL, domain, constraint, and index usage
- [ ] Run migration scripts
- [ ] Test all functionality
- [ ] Update documentation

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
