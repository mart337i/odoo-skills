# OCA Module Migration

End-to-end workflow for migrating an OCA addon/module from one Odoo major version branch to another.

## Scope

This guide covers normal OCA module migration and `[MIG]` pull request preparation. It does not cover OpenUpgrade: do not adopt OpenUpgrade's operating model, its commands, or its workflow for a normal OCA `[MIG]` port.

The OCA version guides cover **framework changes** only, and say so themselves. For data-model changes of each core module, the guides point at the `upgrade_analysis.txt` files inside the OpenUpgrade project: <https://github.com/OCA/OpenUpgrade>. Reading that file for data-model deltas is part of the OCA protocol; running an OpenUpgrade migration is not.

## Before Migrating

The OCA protocol opens every version guide with a short preamble. Follow it:

- Update yourself with the latest OCA Conventions: <https://odoo-community.org/page/contributing>
- Subscribe to the mailing list of the related project: <https://odoo-community.org/groups>
- On the corresponding GitHub issue named `Migration to version <target>`, announce which module(s) you want to migrate
- Install `pre-commit`

The issue announcement is the documented OCA step, not an optional courtesy. Treat it as part of the community workflow when the user wants PR-ready output, and confirm the issue title and repository with the user before posting.

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
4. Announce or confirm work on the related OCA migration issue when the user wants PR-ready community workflow. This is the documented OCA step, not an optional courtesy; see [Before Migrating](#before-migrating).
5. Create a migration branch from the target branch, named like `<target>-mig-<module>`.
6. Apply source-branch module commits onto target branch using the OCA patch flow.
7. Resolve conflicts without dropping history unless the user explicitly chooses a different strategy.
8. Run the pre-commit pass described in [Mechanical Cleanup Pass](#mechanical-cleanup-pass) before the semantic migration fixes.
9. Commit the mechanical formatting separately as `[IMP] $module: pre-commit auto fixes`, using `--no-verify` only for this OCA mechanical-formatting exception and only after the user confirms it.
10. Update the module for the target version, applying every intermediate version checklist.
11. Bump `__manifest__.py` version to `<target>.1.0.0` unless repo policy says otherwise.
12. Remove the `migrations` folder carried over from the previous version.
13. Update imports, XML, views, assets, manifest data, docs, tests, and security as required.
14. Discover likely Odoo install/update/test commands from repo docs/config, `$ODOO_TOOL_README`, and `$ODOO_BASE_COMMAND`, then ask the user before running them.
15. Run agreed verification and fix failures.
16. Commit as `[MIG] $module: Migration to <target>`.
17. After explicit user confirmation, push the migration branch to the contributor's fork and open the pull request, using the title `[<target>][MIG] <module>: Migration to <target>`. See [Delivery To The OCA Fork](#delivery-to-the-oca-fork).

## Command Safety

- Do not add remotes, push branches, post issue comments, or create pull requests without explicit user confirmation.
- Do not run `git filter-branch` or `git filter-repo` without explicit user approval; both rewrite history.
- Do not use `--no-verify` as a general workaround. In this guide it is only the OCA-documented first mechanical-formatting commit exception.
- Before any command that touches an Odoo database or local service, discover likely commands from repo docs/config, `$ODOO_TOOL_README`, and `$ODOO_BASE_COMMAND`, then ask the user.

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

Do not use destructive git commands unless explicitly approved by the user. `git filter-branch` and `git filter-repo` rewrite the whole history, so they always need explicit user approval.

## Module Rename Requires Rewriting History

If the **module is renamed** in the migration, the whole commit history has to be renamed too. This is not cosmetic: if the history is not renamed, then on the next version, when the standard patch flow is executed again, only the latest commit will be brought over. The earlier ones will be silently dropped, and the module history in OCA will fragment across the rename.

These commands rewrite history. Explain the impact, confirm the old and new module names, and get explicit user approval before running either variant.

On older versions of git:

```bash
git filter-branch --tree-filter 'if [ -d $module ]; then mv $module $new_module_name; fi' HEAD
git rebase origin/$target
```

On newer ones:

```bash
git filter-repo --path-rename $module/:$new_module_name/ --refs HEAD --force
```

## Mechanical Cleanup Pass

Run pre-commit/formatting before the semantic migration fixes. The OCA protocol does all the formatting in one dedicated commit the first time, with `--no-verify` so the commit hook does not re-run the linters on the whole tree. Treat this as a narrow OCA exception and ask before using it:

```bash
pre-commit run -a  # to run pre-commit linters and formatters (please ignore pylint errors at this stage)
git add -A
git commit -m "[IMP] $module: pre-commit auto fixes" --no-verify  # it is important to do all the formatting in one commit the first time
```

On this last step pre-commit may change nothing, so just ignore the message saying `Nothing to commit`.

For the 13.0 guide the same step reads `pre-commit run -a  # to run black, isort and prettier (ignore pylint errors at this stage)` with commit message `[IMP] $module: black, isort, prettier`, and for 16.0 the commit message is `[IMP] $module: pre-commit stuff`. The mechanics are otherwise identical.

## Delivery To The OCA Fork

The migration is not finished until it reaches the OCA repository. `$user_org` below is the contributor's GitHub login or organization name.

Commit the migration:

```bash
git add --all
git commit -m "[MIG] $module: Migration to $target"
```

Add the fork as a remote and push the migration branch:

Only run these commands after the user confirms the contributor fork name, remote name, authentication method, and that pushing is intended.

```bash
git remote add $user_org git@github.com:$user_org/$repo.git  # This mode requires an SSH key in the GitHub account
# ... or ...
git remote add $user_org https://github.com/$user_org/$repo.git  # This will required to enter user/password each time
git push $user_org $target-mig-$module --set-upstream
```

Then create the pull request from the fork, following the link the push command prints, or see [Creating a pull request from a fork](https://docs.github.com/en/free-pro-team@latest/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request-from-a-fork).

For being easily searchable and avoiding to duplicate efforts, name the pull request following this pattern: `[<target>][MIG] <module>: Migration to <target>`.

## Things Not To Do

- Do not change copyright years.
- Do not change original authors.
- Do not mix unrelated feature changes into the migration.
- Do not keep migration scripts carried over from the previous version. The OCA guide is unconditional: remove the `migrations` folder inside the module if it exists. If a new target-version upgrade script appears genuinely required, raise it on the migration issue instead of deciding locally.
- Do not adopt the OpenUpgrade operating model, its commands, or its workflow for a normal `[MIG]` port.

## Multi-Version Jumps

**REMARK 2** from the OCA guides: if you are doing a migration jumping several versions, check all the tasks in the successive `Migration to version XX.0` guides until arriving at the target one.

For a jump across multiple versions, apply each target checklist in order.

Examples:

- `14.0` to `17.0` means apply `15.0`, then `16.0`, then `17.0`.
- `18.0` to `20.0` means apply `19.0`, then `20.0`.
- `17.0` to `20.0` means apply `18.0`, then `19.0`, then `20.0`.

## Version Checklists

Each section below is derived from the official OCA guide for that version, at
`https://github.com/OCA/maintainer-tools/wiki/Migration-to-version-<version>`. When this file and
the wiki disagree, the wiki wins.

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
- Remove references in `CREDITS.rst` of past financed migrations by other companies.
- If you are comfortable with squashing commits in Git, you could consider to squash administrative commits (if any) with the previous commit for reducing commit noise. Check [Merge-commits-in-pull-requests](https://github.com/OCA/maintainer-tools/wiki/Merge-commits-in-pull-requests#mergesquash-the-commits-generated-by-bots-or-weblate) for details.
- Replace `tree` view type with `list` in Python, JS, and XML, but avoid renaming XML IDs solely because they contain `tree`.
- Replace context key `tree_view_ref` with `list_view_ref`.
- Replace `user_has_groups` with `self.env.user.has_group`.
- Replace `check_access_rights()` and `check_access_rule()` with `check_access()`.
- Replace `_filter_access_rules()` and `_filter_access_rules_python()` with `_filtered_access()`.
- Replace `_name_search` with `_search_display_name`.
- Replace `_check_recursion()` with `_has_cycle()`.
- Review `copy` and `copy_data`; they now work on multiple records.
- Replace field `group_operator` with `aggregator`.
- Be careful with searches on non-stored related fields: they don't log a warning anymore, they throw an exception.
- Override `search_fetch` instead of `search` when modifying search results where applicable.
- Consider `path` on `ir.actions.act_window` for nicer URLs.
- Replace `<div class="oe_chatter">...` with `<chatter />`.
- Missing fields used in expressions (`invisible`, `column_invisible`, `readonly`, `required`, `context`, `domain`) are now added to the views as invisible ones automatically. If you encounter fields in views with `invisible="True"` or `invisible="1"`, they can probably be removed for cleanliness, as they are auto-added. Reference: [odoo/odoo#137031](https://github.com/odoo/odoo/pull/137031).
- Simplify kanban views: `kanban-box` becomes `card`, fields can be regular field definitions, and obsolete tooltip/colorpicker patterns should be updated.
- Remove `/** @odoo-module **/` from JS files where the version/repo convention allows it.
- Replace `from odoo import registry` with `from odoo.modules.registry import Registry`.
- Prefer `self.env._` for translations where useful.
- If the module contains any company dependent field, a migration script is required for the OpenUpgrade path. Example: [OCA/sale-workflow#3958](https://github.com/OCA/sale-workflow/pull/3958/files). This is the one exception in this section to the "remove previous-version migration scripts" rule, and it applies to the OpenUpgrade path only; a normal OCA `[MIG]` port still does not adopt it, see [Scope](#scope).
- Replace JS tour `extra_trigger` with a separate step using that trigger.
- On JS test tours, remove `test: true,` from the JS definition. Example: [OCA/mass-mailing#29](https://github.com/OCA/mass-mailing/pull/29/changes).
- Add tests to increase code coverage.
- Disable tracking by setting `tracking_disable` on the whole test env, e.g. `cls.env = cls.env(context=dict(cls.env.context, tracking_disable=True))`, or evaluate the use of `BaseCommon` as base test class if you need to setup company, currency, users and groups in a special way.

### 19.0

- Bump module version to `19.0.1.0.0`.
- Remove previous-version migration scripts.
- Remove references in `CREDITS.rst` of past financed migrations by other companies.
- If you are comfortable with squashing commits in Git, you could consider to squash administrative commits (if any) with the previous commit for reducing commit noise. Check [Merge-commits-in-pull-requests](https://github.com/OCA/maintainer-tools/wiki/Merge-commits-in-pull-requests#mergesquash-the-commits-generated-by-bots-or-weblate) for details. See also the 13.0 bullet in this file for the Weblate warning about not squashing translator-authored commits.
- On `res.groups`, the `category_id` field disappears, replaced by the new field `privilege_id`, which points to a new model `res.groups.privilege`, that you may need to create as well if you are defining a new permission section. More info at [odoo/odoo@33637d13](https://github.com/odoo/odoo/commit/33637d137ed52f5faabaafd5c208e187042be8eb).
- Replace `groups_id` with `group_ids` on views, menus, actions, reports, users, and related technical records where applicable. Specifically, the technical models with this change are `res.users`, `ir.ui.view`, `ir.ui.menu`, `ir.actions`, `ir.actions.report` and `website.page.properties`. More info at [odoo/odoo#179354](https://github.com/odoo/odoo/pull/179354).
- Replace `self._cr`, `self._uid`, and `self._context` with `self.env.cr`, `self.env.uid`, and `self.env.context`.
- Replace `odoo.osv.expression` usage with `odoo.fields.Domain` expressions. More info at [odoo/odoo#217708](https://github.com/odoo/odoo/pull/217708).
- Replace `_sql_constraints` with `models.Constraint` and indexes with `models.Index` where applicable. More info at [odoo/odoo#175783](https://github.com/odoo/odoo/pull/175783).
- Replace manual timezone handling through context/pytz with `self.env.tz` where applicable. Check more details in [odoo/odoo#221541](https://github.com/odoo/odoo/pull/221541).
- Replace field parameter `auto_join` with `bypass_search_access`. More info at [odoo/odoo#219627](https://github.com/odoo/odoo/pull/219627).
- Replace `read_group` with `_read_group` for backend processes or `formatted_read_group` for public calls. More info at [odoo/odoo#163300](https://github.com/odoo/odoo/pull/163300).
- Replace controller route `type="json"` with `type="jsonrpc"`. More info at [odoo/odoo#183636](https://github.com/odoo/odoo/pull/183636).
- Search methods for non-stored computed fields should return a `Domain` object instead of a list domain. More info at [odoo/odoo@4f0d4670ed](https://github.com/odoo/odoo/commit/4f0d4670edea0e989fc939e3b2e75cefd351cfc3).
- If you are overriding any of these search methods, and want to introspect the arguments of the search (`operator` and `value`), take into account that the domain passed in the original search may not match, because the domain optimizer may change it. Example: having a search over a boolean field this way, `model.search([("boolean_field", "=", True)])`, the method `_search_boolean_field` will receive as operator `in` (not `=`) and as value `OrderedSet([True])`, not `True`.
- In general, you can replace the use of traditional domains with lists by `Domain` objects (in searches, `filtered_domain`, etc), for clearer syntax and better optimization. Example: [OCA/commission#646](https://github.com/OCA/commission/pull/646/commits/c6a99353a7caf383350640a5ee28ca586cd14e46#diff-01f804f51f9fa191855019e02883f8d65c56f4cd2394fbf6687d6b211d0c2be5L17-R26).
- Replace `toggle_active` usage with explicit `action_archive` or `action_unarchive` where appropriate. More info at [odoo/odoo#183691](https://github.com/odoo/odoo/pull/183691).
- Consider `@api.private` for methods with no initial underscore that should not be RPC-accessible. More info at [odoo/odoo#195402](https://github.com/odoo/odoo/pull/195402).
- You may now use new dynamic dates parameters for domains. More info at [odoo/odoo#216665](https://github.com/odoo/odoo/pull/216665).
- Import `SUPERUSER_ID` from `api` instead of directly from `odoo`. More info at [odoo/odoo@d6a955f10483](https://github.com/odoo/odoo/commit/d6a955f10483).
- Replace `@ormcache_context` with `@ormcache` and explicit context-key parameters. More info at [odoo/odoo#220725](https://github.com/odoo/odoo/pull/220725).
- Replace `urllib.parse.urljoin` with `odoo.tools.urls.urljoin` where applicable. More info at [odoo/odoo@977e62d91f3e](https://github.com/odoo/odoo/commit/977e62d91f3e).
- You may want to use the new `odoo.domain` and `odoo.Domain` API for domain manipulation. See some examples at [odoo/odoo#170009](https://github.com/odoo/odoo/pull/170009).
- Replace `name_search` override parameter `args` with `domain`. More info at [odoo/odoo@e35bcedab2](https://github.com/odoo/odoo/commit/e35bcedab20048807aa3aa17dd8d1e4ea932e3d5).
- Remove decorator `@api.returns`, and adapt any call relying on the former behavior. More info at [odoo/odoo#182709](https://github.com/odoo/odoo/pull/182709). This is a separate 19.0 change from the 13.0 batch removal of `@api.multi`, `@api.returns`, `@api.one`, `@api.cr`, and `@api.model_cr`; both apply when migrating across those versions.
- Remove `string` and `expand` attributes from the `<group>` tags in search views, like done in [odoo/odoo#220023](https://github.com/odoo/odoo/pull/220023).
- Add tests to increase code coverage.
- Disable tracking by setting `tracking_disable` on the whole test env, e.g. `cls.env = cls.env(context=dict(cls.env.context, tracking_disable=True))`, or evaluate the use of `BaseCommon` as base test class if you need to set up company, currency, users and groups in a special way.
- Tests should not rely on demo data being installed by default; create needed test data directly. Demo data is no longer installed by default, and Odoo advises not to rely on it.
- If you were using the `odoo-test-helper` library for loading some fake model in the tests, you can now do it natively without the library.

  Before:

  ```python
  from odoo_test_helper import FakeModelLoader

  @classmethod
  def setUpClass(cls):
      ...
      cls.loader = FakeModelLoader(cls.env, cls.__module__)
      cls.loader.backup_registry()
      from .fake_models import FakeModel
      cls.loader.update_registry((FakeModel,))

  @classmethod
  def tearDownClass(cls):
      cls.loader.restore_registry()
      ...
  ```

  Now:

  ```python
  from odoo.orm.model_classes import add_to_registry

  @classmethod
  def setUpClass(cls):
      ...
      from .fake_models import FakeModel
      add_to_registry(cls.registry, FakeModel)
      cls.registry._setup_models__(cls.env.cr, ["fake.model"])
      cls.registry.init_models(cls.env.cr, ["fake.model"], {"models_to_check": True})
      cls.addClassCleanup(cls.registry.__delitem__, "fake.model")
  ```

- Check tasks of previous versions if you are migrating from lower versions than 18. It's also recommended to check past migration guides for things not done in previous migrations.

### 20.0

#### Official OCA 20.0 Guide Items

The OCA guide for 20.0 is deliberately short. It has exactly four tasks
(<https://github.com/OCA/maintainer-tools/wiki/Migration-to-version-20.0>, lines 14-17):

1. Bump module version to `20.0.1.0.0`.
2. Remove any possible migration script from previous version (in a nutshell, remove the
   `migrations` folder inside the module if it exists).
3. Remove references in `CREDITS.rst` of past financed migrations by other companies.
4. If you are comfortable with squashing commits in Git, you could consider to squash
   administrative commits (if any) with the previous commit for reducing commit noise. Check
   [Merge-commits-in-pull-requests](https://github.com/OCA/maintainer-tools/wiki/Merge-commits-in-pull-requests#mergesquash-the-commits-generated-by-bots-or-weblate)
   for details.

The 20.0 guide has no framework-change list, unlike 16.0 through 19.0. That is the state of the
official protocol, not an omission in this file.

#### Author Guidance, Not From The OCA Wiki

Everything in this subsection is **author guidance, not from the OCA wiki**. It is written for
the case where the wiki is silent and the migration still has to be done. The target Odoo 20
source and the wiki remain authoritative wherever they say something different.

- Confirm that the OCA repository has a `20.0` target branch and that the migration issue or PR context is available before starting.
- Use Python 3.12 through 3.14 within the Odoo 20-supported range and PostgreSQL 16 or newer for local verification.
- Re-run the 19.0 checklist against the Odoo 20 source; do not assume that a 19.0-compatible API, XML ID, asset, or frontend pattern remains unchanged.
- Review model declarations using `models.Constraint`, `models.Index`, `models.UniqueIndex`, `fields.Domain`, `search_fetch`, and `fetch` against the target source and module behavior.
- Review raw SQL for ORM alternatives, `odoo.tools.SQL`, required flushes, cache invalidation, and computed-field notifications.
- Verify list/form/search/kanban view syntax, direct conditional attributes, QWeb templates, report templates, and website/portal inheritance against Odoo 20.
- Verify manifest dependencies, asset bundles, JavaScript module conventions, OWL components, registries, services, and templates against Odoo 20 web source.
- Re-check controllers, JSON/HTTP route types, CSRF behavior, portal ownership, public visibility, `sudo()` boundaries, and multi-company access.
- Add or update ORM, access, HTTP, website, JavaScript, asset, and browser-tour tests at the behavior seam where the module uses them.
- Keep compatibility changes separate from feature changes, preserve authorship/copyright, and update translations and documentation.
- Verify CI, pre-commit, module loading, and the final `[20.0][MIG] <module>: Migration to 20.0` PR title.

The `migrations` folder rule above has no exception in the wiki. If you believe a new upgrade
script is genuinely required for a 20.0 migration, that is a deviation from the OCA protocol and
needs to be raised on the migration issue, not decided locally. For reference, when an upgrade
script is genuinely unavoidable, the Odoo contract is `migrate(cr, version)` under
`migrations/<full-module-version>/` or the repository's established `upgrades/` convention, with
`pre-`, `post-`, or `end-` prefixes and lexical ordering when several scripts share a phase. Use
`odoo.upgrade.util` only when the repository and target execution environment explicitly support
it.

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
