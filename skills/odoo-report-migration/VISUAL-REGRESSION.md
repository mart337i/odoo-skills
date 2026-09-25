# Report Visual Regression Workflow

## Evidence Levels

Use the strongest available evidence and state what was actually checked:

1. **Rendered PDF and HTML**: preferred minimum evidence.
2. **Rasterized page comparison**: useful for geometry, colors, fonts, and clipping.
3. **Text and structural comparison**: useful for values, order, missing sections, and page count.
4. **Manual review only**: acceptable when tooling is unavailable, but record the reviewer and fixture set.

Do not treat PDF byte equality as visual equality. Metadata, timestamps, object
ordering, and renderer internals can change without a visible difference.

## Comparison Procedure

1. Render the source and target using equivalent fixture data and context.
2. Normalize filenames and remove known non-visual metadata from comparison reports.
3. Compare page count, page dimensions, text extraction, and key labels/totals.
4. Rasterize corresponding pages when a PDF renderer is available.
5. Inspect an overlay or side-by-side view for margins, line wrapping, fonts,
   colors, images, table borders, page breaks, and clipping.
6. Compare HTML separately for missing nodes, changed classes, escaping, and
   asset URLs.
7. Classify each difference and either fix it, document it, or obtain a product
   decision that the difference is intentional.

## Useful Assertions

- same report action and output type
- same number of documents and pages, unless a documented renderer difference exists
- same paper width, height, orientation, and effective margins
- same headings, labels, totals, and translated strings
- same header/footer presence and page numbering
- same table columns, row ordering, and repeated header behavior
- no clipped text, overlapping elements, broken images, or missing fonts
- no unexpected blank pages or orphaned totals/signatures
- same access outcome for allowed and denied users

## Reproducibility Record

Keep a small manifest beside comparison artifacts containing:

- source and target Odoo versions
- module and dependency revisions
- database fixture or record description
- user, company, language, timezone, currency, and paper format
- renderer version and relevant command/configuration
- date of render and known non-visual differences

The manifest is more valuable than an unexplained screenshot because it lets a
future migration reproduce the comparison.
