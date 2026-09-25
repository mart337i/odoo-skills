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

## Existing Repository Migration Notes

The repository's OCA migration guide records these relevant checks:

- **16.0**: move XML assets into regular asset bundles instead of
  `web.assets_qweb`, and adapt reports, QWeb, and website markup for Bootstrap 5
  where applicable.
- **17.0**: remove obsolete `owl="1"` from OWL templates where applicable and
  review version-specific template conventions.
- **18.0**: review the broader view and frontend changes in addition to report
  templates; do not assume a report's surrounding templates stayed unchanged.
- **19.0**: review `groups_id` to `group_ids` changes on reports and related
  technical records where applicable.
- **20.0**: verify report behavior against the target source and documentation;
  do not infer unsupported report details from earlier guides.

These notes are intentionally conservative. The target source, module
dependencies, and rendered output remain authoritative.

## Report-Specific Diff Table

For each report, record the result of each check:

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
| Value provider |  |  |  |
| Translations and RTL |  |  |  |
| Renderer |  |  |  |
| Access behavior |  |  |  |
