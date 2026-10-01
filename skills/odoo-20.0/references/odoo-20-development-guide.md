# Odoo 20 Development Guide

Guide for developing Odoo 20 modules: creating modules, manifest, structure, and common patterns.

## Table of Contents
- [Module Structure](#module-structure)
- [Runtime and Tooling](#runtime-and-tooling)
- [Creating a Module](#creating-a-module)
- [Manifest File](#manifest-file)
- [Models](#models)
- [Views](#views)
- [Security](#security)
- [Data Files](#data-files)
- [Assets](#assets)
- [Wizards](#wizards)

---

## Module Structure

### Standard Structure

```
my_module/
├── __init__.py
├── __manifest__.py
├── models/
│   ├── __init__.py
│   └── my_model.py
├── views/
│   └── my_model_views.xml
├── security/
│   ├── ir.access.csv
│   └── my_module_security.xml
├── data/
│   └── my_module_data.xml
├── demo/
│   └── demo_data.xml
├── migrations/
│   └── 20.0.1.0/
│       └── post-migration.py
├── tests/
│   ├── __init__.py
│   └── test_my_model.py
├── wizard/
│   ├── __init__.py
│   └── my_wizard.py
├── controllers/
│   ├── __init__.py
│   └── my_controller.py
├── static/
│   ├── src/
│   │   ├── js/
│   │   ├── xml/
│   │   └── scss/
│   └── description/
│       └── icon.png
└── report/
    └── my_report.xml
```

---

## Runtime and Tooling

Odoo 20 raises the development baseline to Python 3.12+ and PostgreSQL 16+. Update local virtualenvs, containers, CI images, and test databases before investigating module-level failures.

Dependency notes for migrated addons:

- Add explicit external dependencies for packages no longer provided by Odoo packaging, such as `pytz` or `xlwt`, when the addon still requires them.
- Prefer Python's `zoneinfo` for new timezone code.
- Reusing Odoo 20's `ruff.toml` means linting against Python 3.12 and stricter relative-import checks.

Development command changes to check in wrappers and Docker scripts:

- Odoo 20 binds HTTP to localhost by default; containers and VMs may need `--http-interface=0.0.0.0`.
- `--stop` is available as an alias for `--stop-after-init`, but explicit flags are clearer in docs and CI.
- Review new or changed options such as `--unsafe-policy`, `--log-config`, `--db-system`, and `--gevent-workers` before copying older command templates.
- `odoo-bin populate` behavior is addon-backed; scripts using `populate --factors` may need the `populate` module and its dependencies.
- Package Dockerfiles moved under `setup/docker/`; update custom packaging wrappers that referenced old `setup/package.df*` paths.

---

## Creating a Module

### Step 1: Create Directory

```bash
mkdir -p my_module/models
mkdir -p my_module/views
mkdir -p my_module/security
mkdir -p my_module/static/src/js
```

### Step 2: Create __init__.py

```python
# __init__.py

from . import models
from . import controllers
```

```python
# models/__init__.py

from . import my_model
```

### Step 3: Create Model

```python
# models/my_model.py

from odoo import models, fields

class MyModel(models.Model):
    _name = 'my.model'
    _description = 'My Model'

    name = fields.Char(string="Name", required=True)
    description = fields.Text(string="Description")
    active = fields.Boolean(string="Active", default=True)
```

### Step 4: Create Manifest

```python
# __manifest__.py

{
    'name': 'My Module',
    'version': '1.0.0',
    'category': 'Tools',
    'summary': 'My awesome module',
    'description': """
        My Module
        ==========
        This module does something useful.
    """,
    'author': 'Your Name',
    'website': 'https://github.com/yourname/my_module',
    'license': 'LGPL-3',
    'depends': ['base'],
    'data': [
        'security/my_module_security.xml',
        'views/my_model_views.xml',
    ],
    'demo': [
        'demo/demo_data.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'my_module/static/src/js/my_script.js',
        ],
    },
    'installable': True,
    'application': False,
}
```

---

## Manifest File

### Required Fields

```python
{
    'name': 'My Module',      # Required
    'version': '1.0',          # Optional
    'depends': ['base'],       # Optional but recommended
}
```

### Common Fields

```python
{
    # Information
    'name': 'My Module',
    'version': '1.0.0',
    'category': 'Tools',
    'summary': 'Short description',
    'description': 'Long description',
    'author': 'Author Name',
    'website': 'https://example.com',
    'license': 'LGPL-3',

    # Dependencies
    'depends': ['base', 'web'],
    'data': ['views/views.xml'],
    'demo': ['demo/demo.xml'],

    # Assets
    'assets': {
        'web.assets_backend': [
            'my_module/static/src/js/file.js',
        ],
    },

    # Other
    'application': False,
    'installable': True,
    'auto_install': False,
}
```

---

## Models

### Basic Model

```python
from odoo import models, fields

class MyModel(models.Model):
    _name = 'my.model'
    _description = 'My Model'
    _order = 'name'

    name = fields.Char(string="Name", required=True)
    code = fields.Char(string="Code")
    description = fields.Text(string="Description")
    active = fields.Boolean(string="Active", default=True)
```

### Model Inheritance (Extension)

```python
class Partner(models.Model):
    _inherit = 'res.partner'

    my_field = fields.Char(string="My Field")
```

### Model Inheritance (Prototype)

```python
class NewModel(models.Model):
    _name = 'new.model'
    _inherit = 'base.model'

    # Inherits all fields and methods from base.model
    my_field = fields.Char(string="My Field")
```

---

## Views

### Create Views

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <!-- List View -->
    <record id="view_my_model_tree" model="ir.ui.view">
        <field name="name">my.model.tree</field>
        <field name="model">my.model</field>
        <field name="arch" type="xml">
            <list string="My Models">
                <field name="name"/>
                <field name="code"/>
                <field name="active"/>
            </list>
        </field>
    </record>

    <!-- Form View -->
    <record id="view_my_model_form" model="ir.ui.view">
        <field name="name">my.model.form</field>
        <field name="model">my.model</field>
        <field name="arch" type="xml">
            <form string="My Model">
                <sheet>
                    <group>
                        <group>
                            <field name="name"/>
                            <field name="code"/>
                        </group>
                        <group>
                            <field name="active"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="Description">
                            <field name="description"/>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <!-- Search View -->
    <record id="view_my_model_search" model="ir.ui.view">
        <field name="name">my.model.search</field>
        <field name="model">my.model</field>
        <field name="arch" type="xml">
            <search string="Search My Models">
                <field name="name"/>
                <field name="code"/>
                <filter string="Active" name="active" domain="[('active','=',True)]"/>
            </search>
        </field>
    </record>

    <!-- Action -->
    <record id="action_my_model" model="ir.actions.act_window">
        <field name="name">My Models</field>
        <field name="res_model">my.model</field>
        <field name="view_mode">list,form</field>
        <field name="help" type="html">
            <p class="o_view_nocontent_smiling_face">
                Create your first my.model!
            </p>
        </field>
    </record>

    <!-- Menu -->
    <menuitem id="menu_my_model_root" name="My Module" web_icon="my_module,static/description/icon.png"/>
    <menuitem id="menu_my_model" name="My Models" parent="menu_my_model_root" action="action_my_model"/>
</odoo>
```

---

## Security

### Access Rights (CSV)

File: `security/ir.access.csv`

```csv
id,name,model_id,group_id/id,operation,domain
access_my_model_user,my.model user,my.model,base.group_user,cru,"[('create_uid', '=', user.id)]"
access_my_model_manager,my.model manager,my.model,my_module.group_my_module_manager,crud,
```

### Access Domains (XML)

Odoo 20 stores record restrictions in the `domain` field of `ir.access` records. Do not copy older `ir.rule` examples into a 20.0 module without verifying compatibility against the target source.

```xml
<odoo>
    <data noupdate="1">
        <!-- User can create/read/write their own records -->
        <record id="access_my_model_user" model="ir.access">
            <field name="name">My Model: user own records</field>
            <field name="model_id" ref="model_my_model"/>
            <field name="group_id" ref="base.group_user"/>
            <field name="operation">cru</field>
            <field name="domain">[('create_uid', '=', user.id)]</field>
        </record>

        <!-- Manager can see all -->
        <record id="access_my_model_manager" model="ir.access">
            <field name="name">My Model: manager sees all</field>
            <field name="model_id" ref="model_my_model"/>
            <field name="group_id" ref="group_my_module_manager"/>
            <field name="operation">crud</field>
        </record>
    </data>
</odoo>
```

---

## Data Files

### Create Records

```xml
<odoo>
    <record id="my_model_1" model="my.model">
        <field name="name">Record 1</field>
        <field name="code">R001</field>
    </record>

    <record id="my_model_2" model="my.model">
        <field name="name">Record 2</field>
        <field name="code">R002</field>
    </record>
</odoo>
```

### noupdate Flag

```xml
<odoo>
    <data noupdate="1">
        <!-- Only loaded on install -->
        <record id="demo_data" model="my.model">
            <field name="name">Demo</field>
        </record>
    </data>

    <!-- Loaded on install and update -->
    <record id="core_data" model="my.model">
        <field name="name">Core</field>
    </record>
</odoo>
```

---

## Assets

### JavaScript

```python
'assets': {
    'web.assets_backend': [
        'my_module/static/src/js/my_widget.js',
        'my_module/static/src/js/my_view.js',
    ],
    'web.assets_frontend': [
        'my_module/static/src/js/frontend.js',
    ],
},
```

### CSS/SCSS

```python
'assets': {
    'web.assets_backend': [
        'my_module/static/src/scss/my_style.scss',
    ],
    'web.assets_frontend': [
        'my_module/static/src/scss/frontend.scss',
    ],
},
```

---

## Wizards

### TransientModel

```python
from odoo import models, fields

class MyWizard(models.TransientModel):
    _name = 'my.wizard'
    _description = 'My Wizard'

    date = fields.Date(string="Date", required=True, default=fields.Date.context_today)
    note = fields.Text(string="Note")

    def action_confirm(self):
        # Do something
        return {'type': 'ir.actions.act_window_close'}
```

### Wizard View

```xml
<record id="view_my_wizard_form" model="ir.ui.view">
    <field name="name">my.wizard.form</field>
    <field name="model">my.wizard</field>
    <field name="arch" type="xml">
        <form string="My Wizard">
            <group>
                <field name="date"/>
                <field name="note"/>
            </group>
            <footer>
                <button name="action_confirm" string="Confirm" type="object" class="btn-primary"/>
                <button string="Cancel" class="btn-secondary" special="cancel"/>
            </footer>
        </form>
    </field>
</record>
```

### Action to Open Wizard

```python
def action_open_wizard(self):
    return {
        'type': 'ir.actions.act_window',
        'name': 'My Wizard',
        'res_model': 'my.wizard',
        'view_mode': 'form',
        'target': 'new',
        'context': {
            'default_date': fields.Date.context_today(self),
        }
    }
```

---

## Common Patterns

### State Field

```python
state = fields.Selection([
    ('draft', 'Draft'),
    'confirmed', 'Confirmed'),
    ('done', 'Done'),
], string='State', default='draft', tracking=True)
```

### Create Default from Context

```python
def default_get(self, fields_list):
    defaults = super().default_get(fields_list)
    if 'field' in fields_list:
        defaults['field'] = self.env.context.get('default_field', 'default')
    return defaults
```

### Name Search

```python
def name_search(self, name='', args=None, operator='ilike', limit=100):
    args = args or []
    if name:
        args = [('name', operator, name)] + args
    return super().name_search(name, args, operator, limit)
```

---

## References

- Based on Odoo 20 best practices
