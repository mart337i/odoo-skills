---
name: odoo-code-review
description: Review Odoo addon code for correctness, security, performance, migrations, tests, and official Odoo coding guidelines. Use when reviewing Odoo modules, diffs, pull requests, or changed files involving models, fields, XML views, data, controllers, reports, OWL/assets, manifests, access rules, record rules, or OCA migrations.
---

# Odoo Code Review

Use this skill for Odoo-specific code review. Findings come first. Focus on bugs, security risks, behavioral regressions, performance problems, migration hazards, and missing tests before style cleanup.

Official Odoo coding guidelines are still covered by [ODOO-CODING-GUIDELINES.md](ODOO-CODING-GUIDELINES.md), but review scope is broader than style.

## First Move

Identify:

- target Odoo version
- addon/module scope
- changed files or diff range
- stable branch vs development branch
- repo-specific conventions and checks
- available local framework/tooling context: `$ODOO_SOURCE`, `$ODOO_TOOL_README`, `$ODOO_BASE_COMMAND`

If a version-specific reference skill exists, consult it before reviewing version-sensitive behavior:

- Odoo 17: `odoo-17.0`
- Odoo 18: `odoo-18.0`
- Odoo 19: `odoo-19.0`
- Odoo 20: `odoo-20.0`

If `$ODOO_SOURCE` is set, inspect local framework code for APIs, XML syntax, assets, test helpers, and version-specific behavior before relying on memory.

## Review Workflow

1. Determine the review target: whole addon, diff, PR, commit range, or specific files.
2. Read the module manifest first, then relevant imports, models, views, security, data, controllers, tests, assets, reports, and docs.
3. Check official style/guideline concerns with [ODOO-CODING-GUIDELINES.md](ODOO-CODING-GUIDELINES.md).
4. Check Odoo behavior risks with the review checklist below.
5. Return findings ordered by severity with file/line references.
6. State checks run, checks not run, and residual review gaps.

## Stable Branch Rule

When reviewing stable branches, do not ask for noisy restyling. Existing file style supersedes generic formatting preferences. Focus on changed code, correctness, security, maintainability, and obvious guideline violations.

In development branches, apply guidelines more broadly to modified code. Recommend restructuring only when it reduces real risk or the user asks for a wider cleanup.

## Odoo Review Checklist

### Models And ORM

- Model names, `_inherit`, and `_inherits` match real domain boundaries.
- Batch operations are used for multi-record flows.
- No `search()`, `browse()`, `create()`, `write()`, or `unlink()` inside loops unless bounded and justified.
- Domains are correct, indexed where relevant, and multi-company-safe.
- `sudo()` is used only at explicit trust boundaries and does not leak records.
- Direct SQL is parameterized and justified.

### Fields, Computes, Constraints, And Decorators

- `@api.depends` lists complete dependencies, including dotted paths where required.
- `@api.constrains` uses supported field names and raises `ValidationError`.
- Stored computed fields are intentional and searchable/groupable when needed.
- Monetary fields use currency fields; relational fields have correct inverse/ondelete behavior.
- Unlink validation uses version-appropriate hooks such as `@api.ondelete` where applicable.

### XML, Views, Actions, And Data

- View inheritance targets stable anchors and uses syntax valid for the target Odoo version.
- Actions, menus, reports, cron jobs, and server actions reference existing XML IDs.
- Manifest data order is load-safe: groups/security XML, ACLs, base data, views, actions, menus.
- Production data, demo data, and `noupdate` reference data are separated correctly.
- User-facing strings are translatable.

### Security And Access

- Persistent models have appropriate `ir.model.access.csv` entries.
- Record rules match realistic users, groups, and company contexts.
- Portal/public controllers explicitly validate identity, ownership, auth mode, CSRF needs, and input.
- Groups, ACLs, record rules, domains, `sudo()`, and multi-company isolation are reviewed together.
- Exceptions use Odoo exception classes: `UserError`, `ValidationError`, `AccessError`, etc.

### Controllers, Reports, OWL, And Assets

- Controllers use correct `auth`, route type, methods, CSRF behavior, and response handling.
- Report rendering checks access before rendering and avoids unsafe context/data leakage.
- Asset bundles include the right JS/XML/SCSS files for the target Odoo version.
- OWL components keep state/props/events clear and clean up listeners or async work.

### Performance, Transactions, And Concurrency

- Large recordsets avoid N+1 queries and per-record writes.
- Aggregations use Odoo-native tools such as `_read_group`/`read_group` where appropriate for the version.
- Database errors that may abort transactions are isolated with savepoints.
- Concurrency-sensitive operations have realistic locking or retry behavior.
- No business method calls `self.env.cr.commit()` unless explicitly required by Odoo framework semantics.

### Tests, Migration, Manifest, And Docs

- Tests cover behavior seams: ORM workflows, access rules, controllers, reports, module updates, or frontend flows.
- Test tags and base classes match repo/Odoo conventions.
- OCA migrations keep compatibility changes separate from behavior changes.
- Manifest dependencies match imports, inherited models, XML refs, data files, and assets.
- Docs and manifest claims are supported by implemented code.

## Optional Scoring

Use scoring only when the user asks for a scored review. Otherwise, findings-first review is preferred.

Suggested weighted score:

```text
total = 0.25*correctness + 0.20*security + 0.15*performance + 0.10*transactions + 0.10*views_data + 0.10*tests + 0.10*maintainability
```

## Output Format

```markdown
## Findings
- Critical: `path/file.py:XX` - issue, why it matters, suggested fix.
- Major: `path/file.xml:XX` - issue, why it matters, suggested fix.
- Minor: `path/file.py:XX` - issue, why it matters, suggested fix.

## Open Questions
- Only include questions that affect review correctness.

## Checks Run
- Commands or manual review scope.

## Residual Risks
- Areas not reviewed or not verifiable from available context.

## Summary
- Brief change-quality summary after findings.
```

If there are no findings, say `No findings` and mention the review scope and residual risks.

## Response Rules

- Cite exact files and lines for every finding.
- Do not bury bugs under style suggestions.
- Do not recommend broad refactors unless tied to a concrete Odoo risk.
- State assumptions instead of guessing.
- If the issue is execution-flow uncertainty, compose with `odoo-code-tracer`.
- If the issue is missing or weak test coverage, compose with `odoo-test-writer`.
