# OCA Module Migration

End-to-end workflow for migrating an OCA addon/module from one Odoo major version branch to another.

## Scope

This guide covers normal OCA module migration and `[MIG]` pull request preparation. It does not cover OpenUpgrade. Do not use OpenUpgrade commands, `upgrade_analysis.txt`, or OpenUpgrade data-model analysis as the workflow.

## Inputs To Detect

- Source version, for example `17.0`.
- Target version, for example `20.0`.
- Repository, for example `sale-workflow`.
- Module path, for example `sale_order_line_sequence`.
- Current branch, remotes, and whether target/source branches exist locally.
- Existing migration issue named like `Migration to version 20.0`.
- Existing tests, pre-commit config, CI config, and Odoo command conventions.

If Odoo framework source or local development setup is needed, inspect the relevant environment variables when set. If they are not set, ask the user to add whichever ones apply:

```bash
export ODOO_SOURCE=/path/to/odoo
export ODOO_BASE_COMMAND="/path/to/odoo/odoo-bin -c /path/to/odoo.conf --addons-path=/path/to/addons"
export ODOO_TOOL_README=/path/to/local-development/README.md
```

Use `$ODOO_SOURCE` for framework code lookup. Use `$ODOO_BASE_COMMAND` as the local base command to adapt for module install/update/test proposals. Use `$ODOO_TOOL_README` for local wrapper/tooling instructions before running project-specific commands.

## End-To-End Workflow

1. Confirm source version, target version, repo, and module.
2. Inspect the target branch module state, if any, and the source branch module history.
3. Verify that the source and target branches exist on the OCA repository before creating migration work.
4. Announce or confirm work on the related OCA migration issue when the user wants PR-ready community workflow.
5. Create a migration branch from the target branch, named like `<target>-mig-<module>`.
6. Apply source-branch module commits onto target branch using the OCA patch flow.
7. Resolve conflicts without dropping history unless the user explicitly chooses a different strategy.
8. Run the OCA-recommended formatter/pre-commit pass first when the target version guide calls for it.
9. Commit mechanical formatting separately when the guide recommends that convention.
10. Update the module for the target version, applying every intermediate version checklist.
11. Bump `__manifest__.py` version to `<target>.1.0.0` unless repo policy says otherwise.
12. Remove obsolete previous-version `migrations/` folders for normal OCA module migration.
13. Update imports, XML, views, assets, manifest data, docs, tests, and security as required.
14. Discover likely Odoo install/update/test commands from repo docs/config, `$ODOO_TOOL_README`, and `$ODOO_BASE_COMMAND`, then ask the user before running them.
15. Run agreed verification and fix failures.
16. Prepare final commit and PR title in OCA format: `[<target>][MIG] <module>: Migration to <target>`.

## Standard OCA Patch Flow

Use this pattern, replacing variables:

```bash
git clone https://github.com/OCA/$repo -b $target
cd $repo
git checkout -b $target-mig-$module origin/$target
git format-patch --keep-subject --stdout origin/$target..origin/$source -- $module | git am -3 --keep
```

If already cloned:

```bash
git remote update
git checkout -b $target-mig-$module origin/$target
git format-patch --keep-subject --stdout origin/$target..origin/$source -- $module | git am -3 --keep
```

## Conflict Handling

If `git am` fails because a patch does not apply:

```bash
git am --abort
git format-patch --keep-subject --stdout origin/$target..origin/$source -- $module | git am -3 --keep --ignore-whitespace
```

Then resolve conflicts, stage the resolution, and continue:

```bash
git add --all
git am --continue
```

Do not use destructive git commands unless explicitly approved by the user.

## Mechanical Cleanup Pass

For modern OCA migrations, run pre-commit/formatting before semantic migration fixes when the version guide says so. Treat formatting as a separate commit if the guide recommends it.

Typical pattern:

```bash
pre-commit run -a
git add -A
git commit -m "[IMP] $module: pre-commit auto fixes"
```

If pre-commit makes no changes, continue without an empty commit.

## Things Not To Do

- Do not change copyright years.
- Do not change original authors.
- Do not mix unrelated feature changes into the migration.
- Do not keep obsolete migration scripts from the previous version unless the repo has a concrete current-version convention requiring them.
- Do not rely on OpenUpgrade guidance for this workflow.

## Multi-Version Jumps

For a jump across multiple versions, apply each target checklist in order.

Examples:

- `14.0` to `17.0` means apply `15.0`, then `16.0`, then `17.0`.
- `18.0` to `20.0` means apply `19.0`, then `20.0`.
- `17.0` to `20.0` means apply `18.0`, then `19.0`, then `20.0`.

## Version Checklists

### 8.0

- Migrate code to the new API.
- Move files into standard `models/`, `views/`, and `data/` subdirectories.
- Move `icon.png` into `static/description/`.
- Update README from the OCA module template.
- Add or update tests to increase coverage.

### 9.0

- Bump module version to `9.0.1.0.0`.
- Remove previous-version migration scripts.
- Continue new ORM API migration where incomplete.
- Replace `select=True` with `index=True`.
- Prefer stable XML selectors such as `name` attributes instead of string selectors.
- Remove redundant `<data>` wrappers in XML files when `noupdate="0"`.
- Update README from the OCA template and add tests.

### 10.0

- Switch `installable` to `True` when applicable.
- Bump module version to `10.0.1.0.0`.
- Replace `openerp` imports with `odoo` imports.
- Rename `__openerp__.py` to `__manifest__.py` if not already done.
- Replace XML root `<openerp>` tags with `<odoo>`.
- Do not use `@api.one` with `@api.onchange`.
- Change `copy()` decorator from `@api.one` to `@api.multi` when still present.
- Replace `base.group_configuration` with `base.group_system`.
- Update sales group XML IDs from `base` to `sales_team` where applicable.
- Keep applying `select=True` to `index=True` and XML selector cleanup where missed.

### 11.0

- Bump module version to `11.0.1.0.0`.
- Remove Python 2 encoding headers when unnecessary.
- Convert Python 3 incompatible code.
- Remove workflows; they disappeared in this version.
- Adapt settings to the general `res.config.settings` model.
- Replace `ir.values`: use action bindings for menus and `ir.default` for defaults.
- Replace `ir.actions.report.xml` with `ir.actions.report`.
- Replace report template refs such as `report.external_layout` with `web.external_layout` and `report.html_container` with `web.html_container`.
- Remove `group_ids` from `ir.config_parameter` records and read/write them with `sudo()` instead.
- Change widget `kanban_state_selection` to `state_selection`.
- Update `ir.cron`: `model` becomes `model_id`; `function` and `args` become `code`.
- Replace tree `colors` with `decoration-*` attributes.

### 12.0

- Bump module version to `12.0.1.0.0`.
- Use OCA README fragments where repo convention expects them.
- Remove previous-version migration scripts.
- Date/datetime fields now return native Python objects; remove unnecessary `from_string` use.
- Use `fields.Date.to_date` or `fields.Datetime.to_datetime` only when converting old-style strings.
- Add `@api.model_create_multi` to `create` overrides when batch creation is possible.
- For `_parent_store = True`, replace `parent_left`/`parent_right` with `parent_path = fields.Char(index=True)`.
- Ensure view `<label>` elements have `for`.
- Ensure search view `<filter>` elements have `name`.
- Ensure tree/list view buttons have `string` for accessibility.
- Convert SASS styles to SCSS.
- Related fields are readonly by default; add `readonly=False` when write behavior is needed.
- Update base import paths to `odoo.addons.base.models` where needed.
- For PDF reports in tests, pass `force_report_rendering=True` when needed.

### 13.0

- Bump module version to `13.0.1.0.0`.
- Remove previous-version migration scripts.
- Squash administrative bot commits when appropriate, but do not squash legitimate translator-authored Weblate commits.
- Remove decorators `@api.multi`, `@api.returns`, `@api.one`, `@api.cr`, and `@api.model_cr`; adapt behavior where needed.
- Ensure non-stored compute methods assign a value in every case.
- Do not rely on stored computed fields being reset when not assigned.
- Multi-company record rules should generally include `['|', ('company_id', '=', False), ('company_id', 'in', company_ids)]`.
- Replace `sudo(user)` with `with_user(user)` while keeping plain `sudo()` when appropriate.
- Replace `track_visibility="..."` with `tracking=True`.
- Remove `oldname` field attributes.
- Remove `view_type` from window action XML.
- Replace action `multi` with `binding_view_types` or equivalent binding fields.
- Replace decimal precision imports with direct `digits="Precision Name"` strings.
- Use `.env.company` for current active company instead of `.env.user.company_id` when that is the intent.
- Replace mail template `format_tz` with `format_datetime` and explicit timezone/format.
- In JavaScript, replace jQuery promises with native `Promise` patterns where applicable.

### 14.0

- Bump module version to `14.0.1.0.0`.
- Remove previous-version migration scripts.
- Add `ondelete` for every `selection_add` value.
- In XML views, dynamic `invisible` and `readonly` expressions should use `attrs`; direct attributes only support simple/static cases in this version.
- Replace XML shortcut tags `<act_window>` and `<report>` with full `<record>` definitions.
- In list views, consider changing icon button `string` tooltips to `title` to avoid visible text taking space.
- Add explicit ACLs for transient models.
- If overriding `_compute_display_name`, reconsider `name_get` or `name_search` depending on desired behavior.
- Remove invalid `size=X` on `fields.Char`.
- `_name_search` returns IDs, not `(id, display_name)` tuples.
- Remove assignments to computed `global` field in record rules.
- Replace `.with_context(force_company=...)` with `.with_company(...)`.
- Use `sudo()` or supported helpers when reading `ir.actions.*`; `_for_xml_id` is useful for action dictionaries.

### 15.0

- Bump module version to `15.0.1.0.0`.
- Remove previous-version migration scripts.
- Check whether module maturity level can be improved.
- Replace `t-raw` with `t-out`; use `markupsafe.Markup` server-side when intentionally outputting safe HTML.
- Replace `t-esc` with `t-out` where practical.
- Access to `ir.model*` objects is restricted; use `sudo()` or supported helpers.
- Convert mail templates from Jinja-style expressions to QWeb body and inline-template fields using `{{ }}` syntax.
- Move JS/SCSS/XML asset declarations into `__manifest__.py` under `assets`.
- Remove the old manifest `qweb` key; use asset bundles.
- If migrating JS to ES modules, use the repository's Odoo-version convention for module headers and file names.
- Replace `SavepointCase` with `TransactionCase`.
- Use `@api.ondelete` instead of overriding `unlink` for deletion prevention where appropriate.
- Move expensive test record setup from `setUp` to `setUpClass` when safe.

### 16.0

- Bump module version to `16.0.1.0.0`.
- Remove previous-version migration scripts.
- Check whether module maturity level can be improved.
- Use `_rec_names_search` when it can replace a `name_search` override.
- Move `groups_id` restrictions from views onto view elements where appropriate.
- If grouped fields are used by domains/defaults/business logic, include an invisible non-grouped copy where needed.
- Add `colspan="2"` to one2many fields outside notebooks when full form width is desired.
- Replace `fields_get_keys()` with `_fields` or `get_views()` depending on need.
- Replace `fields_view_get` with `get_view`.
- Replace `get_xml_id()` with `get_external_id()`.
- Replace deprecated `flush()`/`recompute()` with `flush_model()`, `flush_recordset()`, or `env.flush_all()`.
- Replace deprecated `refresh()`/`invalidate_cache()` with `invalidate_model()`, `invalidate_recordset()`, or `env.invalidate_all()`.
- Use explicit field index types where helpful, for example `btree` instead of `True`.
- Move XML assets into their proper regular asset bundles, not `web.assets_qweb`.
- Adapt reports, QWeb, and website markup for Bootstrap 5 where applicable.
- Check `super()` calls and method override signatures against upstream.
- Prefer `BaseCommon` for tests when it reduces tracking overhead.

### 17.0

- Bump module version to `17.0.1.0.0`.
- Remove previous-version migration scripts.
- If overriding `name_get`, override `_compute_display_name` instead and include needed dependencies.
- Module hooks now receive `env`; convert `pre_init_hook`, `post_init_hook`, and `uninstall_hook` signatures.
- Replace `get_resource_path` with `file_path` and the new argument syntax.
- Replace `active_id` with `id` in view context where appropriate.
- Avoid `active_model` in view context; use explicit model names where appropriate.
- Replace XML `attrs` and `states` with direct Python expressions under `invisible`, `required`, or `readonly`.
- For tree/list invisible fields, use `column_invisible="1"` when hiding the whole column.
- Simplify `res.config.settings` views: `app`, `block`, and `setting` replace older wrapper div patterns.
- Mock external `requests` calls in tests; Odoo blocks them by default.
- Remove `owl="1"` from OWL templates.
- Use `BaseCommon` as a base test class when it fits.

### 18.0

- Bump module version to `18.0.1.0.0`.
- Remove previous-version migration scripts.
- Replace `tree` view type with `list` in Python, JS, and XML, but avoid renaming XML IDs solely because they contain `tree`.
- Replace context key `tree_view_ref` with `list_view_ref`.
- Replace `user_has_groups` with `self.env.user.has_group`.
- Replace `check_access_rights()` and `check_access_rule()` with `check_access()`.
- Replace `_filter_access_rules()` and `_filter_access_rules_python()` with `_filtered_access()`.
- Replace `_name_search` with `_search_display_name`.
- Replace `_check_recursion()` with `_has_cycle()`.
- Review `copy` and `copy_data`; they now work on multiple records.
- Replace field `group_operator` with `aggregator`.
- Override `search_fetch` instead of `search` when modifying search results where applicable.
- Consider `path` on `ir.actions.act_window` for nicer URLs.
- Replace `<div class="oe_chatter">...` with `<chatter />`.
- Simplify kanban views: `kanban-box` becomes `card`, fields can be regular field definitions, and obsolete tooltip/colorpicker patterns should be updated.
- Remove `/** @odoo-module **/` from JS files where the version/repo convention allows it.
- Replace `from odoo import registry` with `from odoo.modules.registry import Registry`.
- Prefer `self.env._` for translations where useful.
- Replace JS tour `extra_trigger` with a separate step using that trigger.

### 19.0

- Bump module version to `19.0.1.0.0`.
- Remove previous-version migration scripts.
- Replace `groups_id` with `group_ids` on views, menus, actions, reports, users, and related technical records where applicable.
- Replace `self._cr`, `self._uid`, and `self._context` with `self.env.cr`, `self.env.uid`, and `self.env.context`.
- Replace `odoo.osv.expression` usage with `odoo.fields.Domain` or the new domain APIs.
- Replace `_sql_constraints` with `models.Constraint` and indexes with `models.Index` where applicable.
- Replace manual timezone handling through context/pytz with `self.env.tz` where applicable.
- Replace field parameter `auto_join` with `bypass_search_access`.
- Replace `read_group` with `_read_group` for backend processes or `formatted_read_group` for public calls.
- Replace controller route `type="json"` with `type="jsonrpc"`.
- Search methods for non-stored computed fields should return a `Domain` object instead of a list domain.
- Replace `toggle_active` usage with explicit `action_archive` or `action_unarchive` where appropriate.
- Consider `@api.private` for methods with no initial underscore that should not be RPC-accessible.
- Import `SUPERUSER_ID` from `api` instead of directly from `odoo`.
- Replace `@ormcache_context` with `@ormcache` and explicit context-key parameters.
- Replace `urllib.parse.urljoin` with `odoo.tools.urls.urljoin` where applicable.
- Replace `name_search` override parameter `args` with `domain`.
- Tests should not rely on demo data being installed by default; create needed test data directly.
- Fake model tests can use native registry helpers instead of `odoo-test-helper` when appropriate.

### 20.0

- Confirm that the OCA repository has a `20.0` target branch and that the migration issue or PR context is available before starting.
- Bump module version to `20.0.1.0.0`, unless the repository uses a different module-version policy.
- Use Python 3.12 through 3.14 within the Odoo 20-supported range and PostgreSQL 16 or newer for local verification.
- Re-run the 19.0 checklist against the Odoo 20 source; do not assume that a 19.0-compatible API, XML ID, asset, or frontend pattern remains unchanged.
- Review model declarations using `models.Constraint`, `models.Index`, `models.UniqueIndex`, `fields.Domain`, `search_fetch`, and `fetch` against the target source and module behavior.
- Review raw SQL for ORM alternatives, `odoo.tools.SQL`, required flushes, cache invalidation, and computed-field notifications.
- Verify list/form/search/kanban view syntax, direct conditional attributes, QWeb templates, report templates, and website/portal inheritance against Odoo 20.
- Verify manifest dependencies, asset bundles, JavaScript module conventions, OWL components, registries, services, and templates against Odoo 20 web source.
- Re-check controllers, JSON/HTTP route types, CSRF behavior, portal ownership, public visibility, `sudo()` boundaries, and multi-company access.
- Add or update ORM, access, HTTP, website, JavaScript, asset, and browser-tour tests at the behavior seam where the module uses them.
- If data migration scripts are required, use the Odoo upgrade-script contract: `migrate(cr, version)` under `migrations/<full-module-version>/` or the repository's established `upgrades/` convention.
- Name upgrade scripts with `pre-`, `post-`, or `end-` prefixes and use lexical ordering when multiple scripts share a phase.
- Use `odoo.upgrade.util` only when the repository and target execution environment explicitly support it; do not turn a normal OCA port into an OpenUpgrade migration.
- Keep compatibility changes separate from feature changes, remove obsolete pre-20 migration folders, preserve authorship/copyright, and update translations and documentation.
- Verify CI, pre-commit, module loading, and the final `[20.0][MIG] <module>: Migration to 20.0` PR title.

## Odoo 20 Source References

- [Odoo 20 source branch](https://github.com/odoo/odoo/tree/20.0)
- [Odoo 20 release metadata](https://raw.githubusercontent.com/odoo/odoo/20.0/odoo/release.py)
- [Odoo 20 ORM reference](https://github.com/odoo/documentation/blob/20.0/content/developer/reference/backend/orm.rst)
- [Odoo 20 upgrade scripts](https://github.com/odoo/documentation/blob/20.0/content/developer/reference/upgrades/upgrade_scripts.rst)
- [Odoo 20 upgrade utilities](https://github.com/odoo/documentation/blob/20.0/content/developer/reference/upgrades/upgrade_utils.rst)

## Verification Checklist

- Manifest version is correct.
- Manifest dependencies match imports, inherited models, XML refs, assets, and external IDs.
- Data files exist and load in safe order.
- Security files cover new persistent/transient models as needed.
- Views load on target version syntax.
- Tests cover migrated behavior or the absence of tests is explicitly called out.
- Pre-commit or equivalent repo formatting passes.
- Odoo install/update/test command was confirmed before running.
- Final PR title follows `[<target>][MIG] <module>: Migration to <target>`.
