# Odoo Report Design Reference

## 1. Discover the Report Contract

Before editing a report, record:

- source and target Odoo version
- report action XML ID and model
- report type: `qweb-html`, `qweb-pdf`, or another supported type
- root template and every `t-call` in the template chain
- external layout and company/website layout dependencies
- paper format, margins, orientation, DPI, and header spacing
- report-specific Python value provider, if present
- report asset bundle and custom fonts, images, or SCSS
- access groups, record rules, and data visibility expectations
- languages, currencies, company configuration, and RTL requirements

Inspect the target version's source before relying on an inherited template.
An XML ID can remain stable while its structure, classes, or asset dependencies
change.

## 2. Stable Template Shape

A typical report has a wrapper, a document loop, an external layout, and a page
boundary. Use the exact wrappers supported by the target version:

```xml
<template id="report_example">
    <t t-call="web.html_container">
        <t t-foreach="docs" t-as="doc">
            <t t-call="web.external_layout">
                <div class="page">
                    <h2 t-out="doc.name"/>
                </div>
            </t>
        </t>
    </t>
</template>
```

This is a shape, not a promise that every version has identical layout
templates. Verify `web.html_container`, `web.external_layout`, layout variants,
and the page boundary in the target source.

For inherited templates:

1. Read the parent template in the target version.
2. Choose an anchor with semantic meaning and low churn.
3. Make the smallest insertion, attribute change, or replacement possible.
4. Confirm the resulting architecture and render output.

Do not use positional XPath expressions or copy a full upstream template merely
because an older version used them.

## 3. Data and QWeb

Keep calculations, sorting, grouping, access checks, and expensive queries in
Python. The template should compose already-authorized values.

Use:

- `t-out` for escaped text and dynamic attributes
- `t-field` when Odoo field formatting is required
- `t-if`, `t-elif`, and `t-else` for display conditions
- `t-foreach` and `t-as` for collections
- `t-call` for reusable template boundaries
- `t-lang` plus a language-aware record context for translated documents

Check these cases explicitly:

- empty or optional values
- long text and long translated labels
- multiple documents in one print action
- negative, zero, rounded, and multi-currency amounts
- missing images, barcodes, or company data
- unauthorized records and fields

Do not use `t-raw` for ordinary user data. If intentional rich HTML is needed,
document its source, sanitization, and trust boundary.

## 4. Print CSS and Assets

Report rendering is not the same as backend or website rendering. Verify which
bundle the target version loads for reports and put report-specific CSS there.

Prefer:

- explicit font family, size, line height, and color values for print-critical text
- local font files with the correct asset declarations
- predictable table widths and column behavior
- explicit `page-break-*` rules where supported by the renderer
- `max-width`, `max-height`, and predictable image dimensions
- classes owned by the module instead of broad global selectors

Be cautious with:

- flexbox or grid features unsupported by the configured PDF renderer
- remote fonts, images, or stylesheets
- viewport units and browser-only JavaScript
- deeply nested tables and percentage widths that interact badly with margins
- CSS copied from the website without checking report assets
- Bootstrap utility classes whose meaning or availability changed between versions

Treat the report asset bundle, CSS framework version, renderer, and paper
format as one compatibility surface.

## 5. Layout and Pagination

Design the first page and continuation pages deliberately. Check:

- header and footer spacing against paper margins
- repeated table headers on page breaks
- rows that must not split
- totals that must remain with their label
- intentional manual breaks and unintended blank pages
- long addresses, descriptions, notes, and product names
- final-page signature blocks and payment details
- PDF page count across realistic data volumes

Avoid fixing overflow by shrinking the entire report first. Identify whether the
cause is a paper format, margin, asset, font metric, table width, or template
structure problem.

## 6. Localization and Branding

Test at least one translated language and one long-label language when the
report is localized. Verify date, time, number, currency, address, and tax
formatting through Odoo field widgets rather than hand-built strings.

Keep company branding configurable where possible. A report should not assume
that a company has a logo, phone number, bank account, or website configured.

## 7. Verification

Use representative fixtures and retain the render inputs. A useful test set
includes one short document, one multi-page document, long text, missing
optional values, multiple records, a translated document, and a different
company or paper format where applicable.

Compare HTML structure and PDF appearance separately. PDF byte equality is not
a reliable visual test because metadata and renderer output can vary.
