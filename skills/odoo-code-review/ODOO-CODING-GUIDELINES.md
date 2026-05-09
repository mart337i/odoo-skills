# Odoo Coding Guidelines Review

Practical checklist based on the official Odoo 19 coding guidelines.

## Review Principles

- Proper code improves readability, maintenance, debugging, reliability, and complexity control.
- On stable branches, preserve existing file style and keep diffs minimal.
- On development branches, apply guidelines to modified code, but avoid broad restructuring unless the file is already under major revision.
- For community modules, prefer module names with an organization/company prefix when creating new modules.

## Module Structure

Standard important directories:

- `data/`: data XML and demo XML where repo convention places them.
- `models/`: model definitions and business logic.
- `controllers/`: HTTP routes.
- `views/`: backend views and QWeb templates.
- `static/`: web assets, usually split into `src/js/`, `src/scss/`, `src/xml/`, `img/`, `lib/`, and tests where relevant.

Optional directories:

- `wizard/`: transient models and their views.
- `report/`: printable reports, SQL-view report models, report actions, and report templates.
- `tests/`: Python tests and related test assets.

File and folder names should contain only lowercase alphanumerics and underscores: `[a-z0-9_]`. Expected permissions are folders `755` and files `644`.

## File Naming

Models:

- Split business logic by main model.
- Name model files after the main model, for example `plant_order.py`.
- Put each inherited Odoo model in its own file, for example `res_partner.py`.
- If there is only one model, its file can match the module name.

Security:

- Access rights go in `security/ir.model.access.csv`.
- Groups go in `security/<module>_groups.xml`.
- Record rules go in `security/<model>_security.xml`.

Views:

- Backend views are split by model and suffixed with `_views.xml`.
- Main menus not tied to a specific action may live in `<module>_menus.xml`.
- Portal/website QWeb templates live in `<model>_templates.xml`.

Data:

- Split by purpose and main model.
- Use `<model>_data.xml` and `<model>_demo.xml` where practical.
- Shared mail data can live in files like `mail_data.xml`.

Controllers:

- Prefer `<module_name>.py` for the main controller.
- `main.py` is an old convention and should not be introduced for new code.
- Inherited controllers can use the inherited module name, for example `portal.py`.

Static files:

- Use meaningful JS, XML, and SCSS file names.
- Put each JS component in its own file when practical.
- Do not link images or libraries from external URLs; copy required assets into the codebase.

Wizards and reports:

- Wizard files use `<transient>.py` and `<transient>_views.xml` in `wizard/`.
- SQL/statistical reports use `<model>_report.py` and `<model>_report_views.xml` in `report/`.
- Printable reports use `<model>_reports.xml` for actions/paperformat and `<model>_templates.xml` for QWeb templates.

## XML Guidelines

Record format:

- Prefer `<record>` notation for records except where Odoo shortcut tags are preferred.
- Place `id` before `model` on `<record>`.
- In `<field>`, place `name` first, then the value or `eval`, then other attributes by importance.
- Group records by model where dependencies allow it.
- Use `<data>` only for non-updatable data with `noupdate="1"`. If the entire file is noupdate, set it on `<odoo>` instead of nesting `<data>`.

Shortcut tags:

- `menuitem` is preferred for `ir.ui.menu` declarations where supported by the target version.
- `template` is preferred for QWeb views requiring only the arch section.
- For version migrations, respect target-version rules; some shortcuts are removed in newer versions.

XML IDs:

- Menu: `<model_name>_menu` or `<model_name>_menu_<purpose>`.
- View: `<model_name>_view_<view_type>` such as `form`, `list`, `kanban`, `search`.
- Main action: `<model_name>_action`.
- Specific action: `<model_name>_action_<detail>`.
- Window action by view: `<model_name>_action_view_<view_type>`.
- Group: `<module_name>_group_<group_name>`.
- Rule: `<model_name>_rule_<group_or_scope>`.

Names:

- View `name` should usually match the XML ID with dots replacing underscores.
- Actions should have real display names because users see them.

Inherited views:

- Use the same local XML ID as the original record where practical; module prefix prevents collision.
- Add `.inherit.<details>` to the view `name` to explain the override purpose.
- New primary views do not need an inherit suffix.

## Python Imports

Import groups should be ordered as:

1. Python standard library and external libraries, one import per line, sorted.
2. Odoo imports, sorted alphabetically or ASCIIbetically as used by the repo.
3. Odoo addon imports, rarely and only when necessary.

Example:

```python
import base64
import re
from datetime import datetime

from odoo import Command, _, api, fields, models
from odoo.fields import Domain
from odoo.tools.safe_eval import safe_eval as eval

from odoo.addons.website.models.website import slug
```

## Python Idioms

- Prefer readability over clever concision.
- Do not use `.clone()`; use `dict(my_dict)` or `list(old_list)`.
- Prefer literal dict creation when setting several values at once.
- Prefer `dict.update()` for related updates.
- Avoid pointless temporary variables.
- Multiple return points are acceptable when they simplify code.
- Know builtins and avoid redundant defaults like `my_dict.get('key', None)`.
- Use list/dict comprehensions when they improve readability.
- Use truthiness for collections: `if records:` instead of `if len(records):`.
- Iterate directly over iterables: `for key in my_dict:`.
- Use `dict.setdefault()` when it simplifies grouped accumulation.
- Add docstrings and comments for methods or tricky code, not for obvious assignments.

## Odoo Programming

Avoid custom generators and decorators unless there is a strong reason. Prefer Odoo API decorators and recordset helpers such as `filtered`, `mapped`, and `sorted`.

Context:

- Do not mutate context directly; use `with_context()`.
- Use context keys carefully because they propagate automatically.
- Prefix custom context keys with the module name when they influence behavior beyond the immediate call.

Extensibility:

- Split methods when they have more than one responsibility.
- Avoid hardcoding business logic that prevents downstream extension.
- Extract override-friendly methods for domains, filters, and business-rule seams only when it improves readability and extension.
- Name methods according to what they provide.

Transactions:

- Never call `cr.commit()` or `cr.rollback()` yourself unless you explicitly created your own cursor.
- Any exceptional manual cursor handling needs an explicit comment explaining why it is correct and safe.
- Manual commits can cause partial commits, broken rollbacks, polluted tests, data loss, and workflow desynchronization.

Exceptions:

- Avoid broad `except Exception` blocks.
- Catch specific exceptions and keep try/except scope narrow.
- If catching framework exceptions and continuing, isolate work with `self.env.cr.savepoint()`.
- Avoid savepoints in tight loops over large batches; they have overhead.

## Translations

Use `self.env._` for translatable static strings.

Good patterns:

```python
_ = self.env._
error = _('This record is locked!')
error = _('Record %s cannot be modified!', record)
error = _('Answer to question %(title)s is not valid.', title=question)
```

Avoid:

- Translating dynamic strings or concatenated strings.
- Formatting before or after translation instead of passing parameters to `_()`.
- Translating field values manually; translatable fields are handled by the framework.
- Hard-to-read punctuation and formatting in translation strings.

For multiple variables, prefer named placeholders such as `%(title)s`. In Odoo translation strings, `%` formatting is preferred over `.format()` for translator friendliness.

## Naming Conventions

Model names:

- Use dot notation and singular forms: `res.partner`, `sale.order`.
- Wizard/transient models use `<related_base_model>.<action>` and should avoid the word `wizard`.
- SQL report models use `<related_base_model>.report.<action>`.

Python names:

- Odoo Python classes use PascalCase.
- Model variables use PascalCase, for example `Partner = self.env['res.partner']`.
- Common variables use lowercase underscores.
- Variables containing record IDs end in `_id` or `_ids`; do not use `partner_id` for a `res.partner` recordset.

Field names:

- Many2one fields end in `_id`.
- One2many and Many2many fields end in `_ids`.

Method names:

- Compute: `_compute_<field_name>`.
- Search: `_search_<field_name>`.
- Default: `_default_<field_name>`.
- Selection: `_selection_<field_name>`.
- Onchange: `_onchange_<field_name>`.
- Constraint: `_check_<constraint_name>`.
- Object action: `action_<name>` and call `self.ensure_one()` when it only supports one record.

Model member order:

1. Private attributes: `_name`, `_description`, `_inherit`, etc.
2. Default methods and `default_get`.
3. Field declarations.
4. SQL constraints and indexes.
5. Compute, inverse, and search methods in field declaration order.
6. Selection methods.
7. Constraint and onchange methods.
8. CRUD and ORM override methods.
9. Action methods.
10. Other business methods.

## JavaScript

For OWL-specific component lifecycle, reactivity, template, props, refs, plugin, or migration checks, use the `odoo-owl` skill. This section covers only general Odoo static-file and JavaScript guideline concerns.

Static organization:

- `static/lib/`: third-party JS libraries, not minified new libraries unless repo policy requires bundled vendor files.
- `static/src/js/`: source JS.
- `static/src/js/tours/`: end-user tutorial tours.
- `static/src/xml/`: JS-rendered QWeb templates.
- `static/src/scss/`: SCSS.
- `static/tests/`: test assets.
- `static/tests/tours/`: test tours.

Coding checks:

- `use strict;` is recommended for JS files when applicable to the target version/module style.
- Use a linter where repo tooling provides one.
- Never add minified JavaScript libraries.
- Use PascalCase for class declarations.

## CSS And SCSS

Formatting:

- Four-space indentation, no tabs.
- Prefer maximum 80 columns where practical.
- Opening brace has one space after the selector.
- Closing brace is on its own line.
- One declaration per line.
- Use meaningful whitespace.

Property order:

- Order properties from outside to inside, starting with position/layout and ending with decoration such as fonts and filters.
- Scoped SCSS variables and CSS variables go at the top of a block, followed by a blank line.

Naming:

- Avoid `id` selectors.
- Prefix classes with `o_<module_name>` or the main route for website modules.
- The webclient can use the generic `o_` prefix.
- Avoid hyper-specific nested names; prefer compact grandchild-style names.

Variables:

- Global SCSS variables use `$o-[root]-[element]-[property]-[modifier]`.
- Scoped SCSS variables use `$-[variable-name]`.
- Mixins and functions use `o-[name]`; function names should use imperative verbs.
- CSS variables are for DOM-contextual adaptation, not global design-system state.
- Avoid defining CSS variables on `:root` in Odoo UI unless the exception is clear.

## Finding Severity

Treat as high severity:

- Manual transaction commits or rollbacks without explicit owned cursor handling.
- Broad exception handling that leaves ORM state undefined.
- Translation misuse that breaks extraction or runtime translation.
- Security file/layout issues that create missing ACLs or unsafe access.
- Version-incompatible XML or API patterns.

Treat as medium severity:

- Non-standard module/file layout that hurts maintainability.
- Poor XML IDs/names that make overrides hard to track.
- Model/member order or naming issues that make code hard to navigate.
- Context keys likely to cause side effects.

Treat as low severity:

- Pure formatting preferences in unchanged stable-branch code.
- Minor import/order/style cleanup with no behavioral risk.
- Broad guideline suggestions that would create noisy diffs.
