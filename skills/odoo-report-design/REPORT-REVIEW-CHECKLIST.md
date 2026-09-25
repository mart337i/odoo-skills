# Report Design Review Checklist

## Discovery

- [ ] Target Odoo version and renderer are known.
- [ ] Report action, model, root template, inherited templates, and value provider are identified.
- [ ] External layout, paper format, report assets, fonts, images, and dependencies are identified.
- [ ] Access groups, record rules, company behavior, and data visibility are unchanged or intentionally changed.

## Template

- [ ] Parent templates were inspected in the target version.
- [ ] XPath anchors are semantic and minimal.
- [ ] Report wrappers and page boundaries match the target version.
- [ ] Business calculations are outside the template where practical.
- [ ] User-provided values use escaped output.
- [ ] Optional values and empty collections render safely.
- [ ] Multiple records render as separate documents.

## Styling

- [ ] Report CSS is loaded by the report asset bundle.
- [ ] Fonts and images are local or have a tested, reliable loading path.
- [ ] Critical dimensions do not depend on unsupported browser behavior.
- [ ] Bootstrap or utility classes are verified for the target version.
- [ ] Tables handle long content, repeated headers, and page breaks.
- [ ] Header, footer, margins, and page numbering fit the paper format.

## Output

- [ ] HTML output was inspected.
- [ ] PDF output was rendered.
- [ ] Short and multi-page fixtures were checked.
- [ ] Long text, missing values, images, barcodes, and totals were checked.
- [ ] At least one translated or RTL case was checked when relevant.
- [ ] The final page does not overflow or create unintended blank pages.
- [ ] Intentional visual differences are documented.
