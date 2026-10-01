# Odoo 20 Security Guide

Guide for Odoo 20 security: access rights with domains, groups, field permissions, and security pitfalls.

## Table of Contents

- [Security Overview](#security-overview)
- [Groups](#groups)
- [Access Rights](#access-rights)
- [Access Domains](#access-domains)
- [Access APIs](#access-apis)
- [Field Access](#field-access)
- [Security Pitfalls](#security-pitfalls)

---

## Security Overview

Odoo 20 consolidates model access and record filtering into access records:

1. **Access Rights** - `ir.access` records grant operations on a model.
2. **Access Domains** - the `domain` on each access record restricts which records that operation applies to.

Both are linked to users through **groups**. Legacy `ir.model.access`/`perm_*` CSVs and `ir.rule` examples are not the Odoo 20 shape; verify existing migrated modules against the target source before copying older snippets.

---

## Groups

`res.groups` - Users belong to groups, security mechanisms are associated to groups.

Use the group and category/privilege model exposed by the target Odoo 20
source and existing module conventions. Do not infer a security model change
from a third-party pattern library; verify XML IDs, fields, and user-form
behavior before changing group definitions.

### Key Attributes

| Attribute      | Description                                        |
| -------------- | -------------------------------------------------- |
| `name`         | User-readable identification (role/purpose)        |
| `privilege_id` | Permission section; replaces direct `category_id`  |
| `implied_ids`  | Other groups implied by this one                   |
| `comment`      | Additional notes                                   |

### Example

```xml
<record id="privilege_my_module" model="res.groups.privilege">
    <field name="name">My Module</field>
    <field name="category_id" ref="base.module_category_my_module"/>
</record>

<record id="group_my_module_user" model="res.groups">
    <field name="name">My Module User</field>
    <field name="privilege_id" ref="privilege_my_module"/>
    <field name="implied_ids" eval="[(4, ref('base.group_user'))]"/>
    <field name="comment">Users can access my module features.</field>
</record>
```

---

## Access Rights

Access rights **grant** operations on a model. If no `ir.access` record grants an operation to the user for the target record, access is denied.

`ir.access` replaces the old `ir.model.access`/`perm_read`/`perm_write`/`perm_create`/`perm_unlink` shape in Odoo 20. A single record uses an `operation` string containing any of `c`, `r`, `u`, and `d`, and may include a record `domain`.

`security/ir.access.csv` - access records.

### Key Attributes

| Attribute   | Description                                                     |
| ----------- | --------------------------------------------------------------- |
| `name`      | Purpose or role of the access record                            |
| `model_id`  | Model technical name in CSV, or model record in XML             |
| `group_id`  | Group granted access; use `base.group_everyone` for all users; leave empty only for global restrictions |
| `operation` | Combination of `c`, `r`, `u`, `d` for create/read/update/delete |
| `domain`    | Optional domain limiting records for this access                |

### Example CSV (Common Format)

File: `security/ir.access.csv`

```csv
id,name,model_id,group_id/id,operation,domain
access_my_model_user,my.model user,my.model,my_module.group_my_module_user,ru,"[('create_uid', '=', user.id)]"
access_my_model_manager,my.model manager,my.model,my_module.group_my_module_manager,crud,
```

### Example XML

```xml
<record id="access_my_model_user" model="ir.access">
    <field name="name">my.model.user</field>
    <field name="model_id" ref="my_module.model_my_model"/>
    <field name="group_id" ref="group_my_module_user"/>
    <field name="operation">ru</field>
    <field name="domain">[('create_uid', '=', user.id)]</field>
</record>
```

---

## Access Domains

Access domains are the Odoo 20 replacement for separate record-rule records in normal module security data. Put record restrictions directly on the relevant `ir.access` row.

An empty `domain` means unrestricted access for that access row. A non-empty domain is evaluated with the same contextual variables used by core access CSVs.

### Domain Variables

| Variable      | Description                                     |
| ------------- | ----------------------------------------------- |
| `time`        | Python's `time` module                          |
| `user`        | Current user (singleton recordset)              |
| `company_id`  | Current user's selected company (single id)     |
| `company_ids` | All companies the user has access (list of ids) |

### Example: Multi-Company Access

```csv
id,name,model_id,group_id/id,operation,domain
my_model_company_access,my.model company,my.model,base.group_everyone,crud,"['|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]"
```

```xml
<record id="my_model_company_access" model="ir.access">
    <field name="name">My Module: multi-company</field>
    <field name="model_id" ref="model_my_model"/>
    <field name="group_id" ref="base.group_everyone"/>
    <field name="operation">crud</field>
    <field name="domain">['|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]</field>
</record>
```

### Example: User-Only Records

```xml
<record id="my_model_user_access" model="ir.access">
    <field name="name">My Module: user records</field>
    <field name="model_id" ref="model_my_model"/>
    <field name="group_id" ref="group_my_module_user"/>
    <field name="operation">ru</field>
    <field name="domain">[('user_id', '=', user.id)]</field>
</record>
```

Group-less `ir.access` rows are for global restrictions, not normal global grants. For access intended for all users, set `group_id/id` to `base.group_everyone`. Treat any group-less row as high-risk: test portal/public/internal users, multi-company users, and negative cases whenever a domain is added or changed.

---

## Access APIs

Odoo 20 access checks are centered on access domains composed from `ir.access` records. Use the new recordset APIs when writing module code:

| API | Purpose |
| --- | ------- |
| `check_access(operation)` | Raise if the recordset is not allowed for `operation` |
| `has_access(operation)` | Return whether the recordset is allowed |
| `_filtered_access(operation)` | Return only records accessible for `operation` |
| `_access_domain(operation)` | Return the domain expressing access for `operation` |
| `has_field_access(field, operation)` | Return whether a field is accessible |
| `check_field_access(field, operation)` | Raise if a field is not accessible |

When migrating from older versions, audit calls to `check_access_rights`, `check_access_rule`, `_filter_access_rules`, `_filter_access_rules_python`, `check_field_access_rights`, and custom `_check_access` overrides. Custom model-level access behavior should usually extend `_access_domain()` so it composes with `ir.access` grants and restrictions.

---

## Field Access

An ORM field can have a `groups` attribute providing a list of groups (comma-separated external identifiers).

If the current user is not in one of the listed groups:

- Restricted fields are removed from views
- Restricted fields are removed from `fields_get()` responses
- Attempts to read/write restricted fields result in an access error

```python
notes = fields.Text('Internal Notes', groups='base.group_system')

# Only users in base.group_system can see/edit this field
```

---

## Security Pitfalls

### Unsafe Public Methods

Any public method can be executed via RPC call. Methods starting with `_` are not callable from action buttons or external API.

```python
# BAD: this method is public and arguments can not be trusted
def action_done(self):
    if self.state == "draft" and self.env.user.has_group('base.manager'):
        self._set_state("done")

# GOOD: keep internal helpers private and enforce authorization in public methods
def _set_state(self, new_state):
    self.sudo().write({"state": new_state})
```

### Bypassing the ORM

**Never** use the database cursor directly when the ORM can do the same thing!

#### Wrong (very bad)

```python
# SQL injection vulnerability
self.env.cr.execute('SELECT id FROM auction_lots WHERE auction_id in (' +
                    ','.join(map(str, ids)) + ') AND state=%s AND obj_price > 0',
                    ('draft',))
```

#### Better (no injection, but still wrong)

```python
self.env.cr.execute('SELECT id FROM auction_lots WHERE auction_id in %s '
                    'AND state=%s AND obj_price > 0',
                    (tuple(ids), 'draft',))
```

#### Best (use ORM)

```python
auction_lots_ids = self.search([
    ('auction_id', 'in', ids),
    ('state', '=', 'draft'),
    ('obj_price', '>', 0)
]).ids
```

#### Using SQL class (recommended)

```python
from odoo.tools import SQL

auction_lots_ids = self.env.execute_query(SQL("""
    SELECT id FROM auction_lots
    WHERE auction_id IN %s AND state = %s AND obj_price > 0
""", tuple(ids), 'draft'))
```

### SQL Injections

**Never** use Python string concatenation (+) or interpolation (%) for SQL queries.

Use psycopg2 parameter passing or `odoo.tools.SQL` wrapper.

```python
# BAD: SQL injection vulnerability
self.env.cr.execute('SELECT distinct child_id FROM account_account_consol_rel '
                    'WHERE parent_id IN ('+','.join(map(str, ids))+')')

# GOOD: use parameter passing
self.env.cr.execute('SELECT DISTINCT child_id '
                    'FROM account_account_consol_rel '
                    'WHERE parent_id IN %s',
                    (tuple(ids),))

# BETTER: use SQL wrapper
from odoo.tools import SQL
self.env.cr.execute(SQL("""
    SELECT DISTINCT child_id
    FROM account_account_consol_rel
    WHERE parent_id IN %s
""", tuple(ids)))
```

### Building Domains

Use `odoo.fields.Domain` to handle domain manipulation safely.

```python
# BAD: the user can pass ['|', ('id', '>', 0)] to access all
domain = ...  # passed by user
security_domain = [('user_id', '=', self.env.uid)]
domain += security_domain  # can have side effects
self.search(domain)

# GOOD: use Domain class
from odoo.fields import Domain
domain = Domain(...)
domain &= Domain('user_id', '=', self.env.uid)
self.search(domain)
```

### Unescaped Field Content

Avoid using `t-raw` for rich-text content - it's an XSS vector.

#### Bad XML

```xml
<div t-name="insecure_template">
    <div id="information-bar"><t t-raw="info_message" /></div>
</div>
```

#### Good XML

```xml
<div t-name="secure_template">
    <div class="info"><t t-out="message" /></div>
    <div class="subject"><t t-out="subject" /></div>
</div>
```

### Escaping vs Sanitizing

**Escaping** (always mandatory) converts TEXT to CODE.

```python
from markupsafe import Markup

# data is TEXT
code = html_escape(data)  # Convert to CODE
self.website_description = Markup("<strong>%s</strong>") % code
```

**Sanitizing** converts CODE to SAFER CODE (but not necessarily safe).

```python
from odoo.tools import html_sanitize

code = Markup("<p class='text-warning'>Important</p>")
html_sanitize(code, strip_classes=True)  # => Markup('<p>Important</p>')
```

### Evaluating Content

Avoid `eval()` - use `ast.literal_eval()` for parsing.

```python
# BAD: very bad
domain = eval(self.filter_domain)

# BETTER: still not recommended
from odoo.tools import safe_eval
domain = safe_eval(self.filter_domain)

# GOOD: use literal_eval
from ast import literal_eval
domain = literal_eval(self.filter_domain)
```

### Accessing Object Attributes

Use `__getitem__` instead of `getattr()` for dynamic field access.

```python
# BAD: unsafe, can access any attribute
def _get_state_value(self, res_id, state_field):
    record = self.sudo().browse(res_id)
    return getattr(record, state_field, False)

# GOOD: safer
def _get_state_value(self, res_id, state_field):
    record = self.sudo().browse(res_id)
    return record[state_field]
```

---

## References

- Source: Odoo 20 documentation `/doc/developer/reference/backend/security.rst`
