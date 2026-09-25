---
name: odoo-report-design
description: Designs and reviews Odoo QWeb HTML and PDF reports with stable layout, print-safe assets, paper formats, localization, and maintainable templates. Use when creating, redesigning, reviewing, or debugging Odoo reports, QWeb report templates, report CSS, headers, footers, tables, page breaks, or PDF rendering.
---

# Odoo Report Design

Use this skill for report presentation and rendering work. Compose it with the
target version skill, `odoo-website` for shared QWeb/website concerns,
`odoo-web-testing` for browser and asset verification, and `odoo-migration` for
major-version ports.

## First Move

Identify the target Odoo version, report action, model, template chain, paper
format, external layout, report asset bundle, and any custom report value
provider. Inspect the target version's parent templates and assets before
choosing XPath anchors or CSS classes.

## Workflow

1. Define the output contract: paper size, orientation, margins, header/footer,
   page numbering, languages, RTL requirements, expected page breaks, and
   whether HTML and PDF must match.
2. Separate data preparation from presentation. Keep business calculations in
   Python and use QWeb for composition, formatting, conditions, and iteration.
3. Build the smallest stable structure around the target version's supported
   report wrappers, external layout, and `.page` boundary.
4. Put report-only CSS and fonts in the target version's report asset bundle.
   Do not assume backend or website assets are available to PDF rendering.
5. Prefer semantic anchors and small inherited changes. Avoid copying core
   templates when an extension is sufficient.
6. Render representative records as HTML and PDF before calling the report
   complete.

## Design Guardrails

- Use escaped output by default: `t-out` for text and `t-field` for formatted
  model values. Treat rich HTML as an explicit, documented trust boundary.
- Keep tables resilient to long names, translated labels, missing values, and
  multi-record output.
- Prefer local, bundled assets and explicit print CSS over remote resources or
  browser-only behavior.
- Treat paper dimensions, margins, DPI, fonts, and renderer availability as
  part of the design, not deployment details.
- Preserve report access rules and data visibility while changing presentation.

## References

- [REPORT-DESIGN.md](REPORT-DESIGN.md) - layout, QWeb, CSS, assets, and print constraints.
- [REPORT-REVIEW-CHECKLIST.md](REPORT-REVIEW-CHECKLIST.md) - implementation and review checklist.
