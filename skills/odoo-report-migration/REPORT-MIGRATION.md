# Odoo Report Migration Reference

## 1. Define Preservation

"Keep the same appearance" must be made testable. Record which properties are
invariants and which may change because of the target platform:

| Area | Examples of invariants |
| --- | --- |
| Content | Same fields, labels, totals, ordering, and conditional sections |
| Geometry | Same paper size, orientation, margins, column widths, and page count where practical |
| Branding | Same logo treatment, colors, fonts, header, footer, and signature blocks |
| Pagination | Same intentional breaks, repeated headers, and total placement |
| Localization | Same language, date, number, currency, address, and RTL behavior |
| Security | Same report availability and record/field visibility |

Document permitted differences, such as a renderer's metadata or a deliberate
target-version accessibility change.

## 2. Capture a Baseline

Use the source version's established Odoo command and test database. Do not run
database-touching commands until the repository setup and user confirmation
requirements are satisfied.

For each fixture, retain:

- report action XML ID and template XML ID
- source version and module commit
- database/module state
- user, company, language, timezone, and currency
- paper format and report context
- record IDs or a reproducible fixture description
- HTML output, PDF output, and render metadata
- screenshots or rasterized pages when available

The fixture set should include short and multi-page documents, long text,
missing optional data, multiple records, translated content, and any custom
company or paper format that matters to the report.

## 3. Inventory Before Editing

Search the module and dependencies for:

- `ir.actions.report` records and bindings
- report root templates and `inherit_id` extensions
- `web.html_container`, external layouts, and layout variants
- `report.paperformat` records and margin/header settings
- `web.report_assets_common` or the target version's report bundle
- CSS, SCSS, fonts, images, barcode URLs, and external resources
- `_get_report_values` and other report model methods
- `t-lang`, translated strings, field widgets, and currency formatting
- report-specific access groups, record rules, and controller routes

Build a dependency map before changing a template. A visual difference can be
caused by a parent layout or asset bundle rather than the report's own XML.

## 4. Inspect the Target

Verify the target version's:

1. report action field names and binding behavior
2. root and external layout template structure
3. report asset bundle and asset declaration syntax
4. CSS framework and utility class behavior
5. supported PDF/HTML renderer and resource-loading behavior
6. paper format defaults and margin interpretation
7. template directives and escaping rules

Use official target source and documentation where local Odoo source is not
available. Never assume that a stable XML ID implies stable descendants or
classes.

## 5. Port in Layers

Port in this order so failures are attributable:

1. Manifest and asset declarations.
2. Report action and binding metadata.
3. Report value provider and data/API compatibility.
4. Root template and external layout integration.
5. Report body structure and QWeb directives.
6. Paper format and print CSS.
7. Fonts, images, barcodes, translations, and optional sections.

After each layer, render at least one representative fixture. Keep migration
compatibility changes separate from intentional redesign.

## 6. Diagnose Differences

Classify a mismatch before changing code:

- data mismatch: values, ordering, formatting, or permissions
- template mismatch: changed parent structure, XPath, or directive behavior
- asset mismatch: missing CSS, fonts, images, or wrong bundle
- geometry mismatch: paper format, margins, DPI, or renderer settings
- CSS mismatch: changed Bootstrap utility or selector behavior
- renderer mismatch: supported CSS, font metrics, pagination, or metadata
- environment mismatch: locale, company configuration, database state, or user

Fix the narrowest cause. Preserve the source structure when it remains valid;
adapt only the parts whose target contract changed.

## 7. Acceptance

A migrated report is ready when:

- the module loads and the report action is available as intended
- source and target fixtures contain equivalent business data
- HTML structure and PDF output were both inspected
- intentional visual invariants are met or documented
- no unexpected page overflow, blank pages, clipping, or missing assets remain
- translations, RTL, multi-record output, and long-content cases pass where relevant
- access and data visibility match the source behavior
- the migration notes identify unresolved renderer limitations
