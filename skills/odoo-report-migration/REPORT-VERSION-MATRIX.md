# Report Migration Version Matrix

This matrix is an audit guide, not a replacement for inspecting the target
Odoo source. Apply every intermediate version checklist for a multi-version
migration.

## Every Version Boundary

- [ ] Confirm report action field names, bindings, groups, and output types.
- [ ] Confirm root, external-layout, and company-layout template structure.
- [ ] Confirm report asset bundle names and manifest/XML asset syntax.
- [ ] Confirm CSS framework version and utility class behavior.
- [ ] Confirm PDF renderer, supported CSS, font loading, and pagination behavior.
- [ ] Confirm paper format defaults and margin/header-spacing interpretation.
- [ ] Confirm QWeb escaping, field widgets, language context, and barcode behavior.

## OCA Wiki Report-Relevant Changes By Version

Extracted from the official OCA migration guides at
`https://github.com/OCA/maintainer-tools/wiki/Migration-to-version-<version>`.
Line references point at that page. Apply every version you cross. Where a
version below has no report-relevant item, the wiki has none: do not invent
one for symmetry.

- **8.0**: no report-relevant item. The 8.0 guide covers only the new API
  port, the `models`/`views`/`data` directory move, `icon.png`, the README
  template, and tests.
- **9.0**: replace string selectors in XML by `name`, or another stable
  attribute selector, whenever possible. For example, change
  `<group string="x" position="after">` by `<group name="x" position="after">`.
  Report template inheritance that positions nodes by a `string` selector
  must move to a `name` selector. (wiki line 17)
- **10.0**: the same `string` to `name` XML selector change (wiki line 24);
  replace the `<openerp>`/`</openerp>` tags in XML files by
  `<odoo>`/`</odoo>`, report data files included (wiki line 26); remove
  `<data>` and `</data>` in XML files if `noupdate="0"` (wiki line 25).
- **11.0**: the `report` module has been split between `base` and `web`
  (wiki lines 23-31):
  - `ir.actions.report.xml` > `ir.actions.report`, as the model has changed.
  - `report.external_layout` > `web.external_layout`.
  - `report.external_layout_header`: no direct equivalent. You need to insert
    the changes inside the `div class="header o_clean_header"` element of the
    `web.external_layout_?` view, being `?` one of the available "themes"
    (`background`, `boxed`, `clean` and `standard` in core).
  - `report.external_layout_footer`: the same as above, but looking inside
    the `<div class="footer o_background_footer">` element.
  - `report.html_container` > `web.html_container`.
  - `report.layout` > `web.report_layout`.
  - `report.minimal_layout` > `web.minimal_layout`.

  Separately, for `ir.values` entries that showed an option under the "Print"
  or "Actions" dropdown: remove the record and just add a field
  `binding_model_id` on the linked `ir.actions.act_window` record, with
  `binding_type` = `report` if the `ir.values` had `key2` =
  `client_print_multi`. Entries that carried defaults go to `ir.default`
  instead. (wiki lines 20-22)
- **12.0**: if you need to generate a PDF in tests, you have to pass the
  context key `force_report_rendering=True`, as `qweb_render_pdf` now
  defaults to `qweb_render_html` in a test environment (wiki line 26).
  Convert SASS styles to SCSS (wiki line 19).
- **13.0**: no change to `ir.actions.report` and no change to report
  templates in the wiki. The one item that reaches reports is mail templates:
  you can't use `format_tz` anymore, use `format_datetime` with explicit tz
  and date format arguments, for example `format_tz(object.date)` becomes
  `format_datetime(object.date, tz=object.tz, dt_format="dd/MM/yyyy HH:mm")`
  (wiki line 34).
- **14.0**: the XML shortcut tags `<act_window>` and `<report>` are no
  longer valid, and must be replaced by their full definition through a
  `<record>` tag. Take into account that the `model` attribute of the
  `<record>` tag was transferred to both `model` and `binding_model_id`
  fields before, so you should put both lines on the new `<record>` tag.
  Example: <https://github.com/odoo/odoo/commit/6835aeb0de6895f7f4d6b23e0b4654465ef21d6a>
  (wiki line 15)
- **15.0**:
  - `t-raw` QWeb directives are replaced by `t-out` ones, with the optional
    Python tool `markupsafe.Markup` for escaping HTML content server side.
    HTML fields apply the markup directly when used on QWeb. (wiki line 14)
  - `t-esc` directives are also deprecated, but still work as an alias of
    `t-out`. When possible, switch to the new name in advance of the future
    directive removal. (wiki line 15)
  - Jinja syntax in mail templates has been replaced by two languages: QWeb
    for the body of the mail template, and `inline_template`, which is
    similar to Jinja but with expressions enclosed by `{{` and `}}` instead
    of `${` and `}`, used in fields like subject, `email_from`, `email_to`.
    (wiki lines 17-21)
  - Assets are now directly declared in the manifest. Move `.js` and `.scss`
    links from templates to `__manifest__.py` under the `assets` key, and
    link all qweb `.xml` in the `web.assets_qweb` bundle. There is no more
    `qweb` key in `__manifest__.py`. The path to the files is now relative
    to the root folder, not the module folder. (wiki lines 23-39)
- **16.0**: XML assets templates are now added into regular asset bundles.
  Previously they were added to the fake asset bundle `web.assets_qweb`, but
  now they should be declared in their proper assets bundle that fits their
  scope (`web.assets_backend`, `web.assets_frontend`, etc). (wiki line 28)
  Reports, QWeb templates and website elements should be adapted if they
  use some Bootstrap 4 elements, as now Odoo is using Bootstrap 5. (wiki
  line 30)
- **17.0**: the XML attributes `attrs` and `states` are no longer used, and
  should be replaced by their equivalent Python expression under
  `invisible`, `required` and/or `readonly`. For example,
  `attrs="{'invisible': [('name', '=', 'red')]}"` becomes
  `invisible="name == 'red'"`. (wiki line 26) The same guide also removes
  `owl="1"` from OWL templates (wiki line 74); that is a JavaScript template
  item rather than a report item, but it lands in the same migration.
- **18.0**: no report-specific item in the wiki. The `tree` to `list` rename
  (wiki line 18) is a view-type change and does not affect
  `ir.actions.report` or report templates. The next report-relevant change is
  19.0's `group_ids`. Do review the broader view and frontend changes around
  the report: do not assume a report's surrounding templates stayed
  unchanged.
- **19.0**:
  - on views, menus, actions and reports, replace the `groups_id` field by
    `group_ids`. The technical models with this change are `res.users`,
    `ir.ui.view`, `ir.ui.menu`, `ir.actions`, `ir.actions.report` and
    `website.page.properties`. (wiki line 19)
  - replace `odoo.osv.expression` by `odoo.fields.Domain` expressions
    (wiki line 21), and replace traditional domain lists with `Domain`
    objects in searches and `filtered_domain` for clearer syntax and better
    optimization (wiki line 29). This reaches report action `domain`
    evaluation and any custom search code the report calls.
  - if you override a search method and introspect its `operator` and
    `value` arguments, the domain optimizer may have changed the domain you
    receive, so the domain passed in the original search may not match
    (wiki line 28).
  - if your tests rely on demo data, it is no longer installed by default, so
    create the records the report needs in the test itself. (wiki line 42)
- **20.0**: no report-relevant item. The 20.0 guide has four tasks, none of
  which touches reports (wiki lines 14-17). Verify report behavior against
  the Odoo 20 source and documentation; do not infer unsupported report
  details from earlier guides.

These notes are intentionally conservative. The target source, module
dependencies, and rendered output remain authoritative. The full version
checklists, including everything that is not report-specific, live in
[odoo-migration](../odoo-migration/OCA-MODULE-MIGRATION.md).

## Report-Specific Diff Table

For each report, record the result of each check. Every checkbox in
"Every Version Boundary" above has at least one row to record into.

| Surface | Source | Target | Result / evidence |
| --- | --- | --- | --- |
| Action and binding |  |  |  |
| Root template |  |  |  |
| External layout |  |  |  |
| Company layout |  |  |  |
| Paper format |  |  |  |
| Report assets |  |  |  |
| CSS framework |  |  |  |
| Fonts and images |  |  |  |
| QWeb escaping |  |  |  |
| Field widgets |  |  |  |
| Value provider |  |  |  |
| Translations and RTL |  |  |  |
| Barcode behavior |  |  |  |
| Renderer |  |  |  |
| Access behavior |  |  |  |
