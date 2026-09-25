---
name: odoo-code-tracer
description: Trace Odoo execution flow from an entry point through controllers, button actions, cron jobs, model methods, overrides, computes, onchanges, constraints, database operations, security checks, and side effects. Use before implementation, during debugging, impact analysis, performance review, or code review when behavior depends on Odoo call chains.
---

# Odoo Code Tracer

Use this skill to map how Odoo code actually executes. Trace only what you can verify in code. If a path is uncertain, mark it as uncertain and explain what evidence is missing.

## First Move

Identify the starting point:

- HTTP route or controller
- form/list button action
- server action or automated action
- cron job
- ORM method called by another addon
- compute, onchange, constraint, inverse, or create/write/unlink hook
- report render
- OWL/web client action calling backend code
- external webhook or integration callback

Detect the target Odoo version. If a version-specific reference exists, consult it before tracing version-sensitive behavior:

- Odoo 17: `odoo-17.0`
- Odoo 18: `odoo-18.0`
- Odoo 19: `odoo-19.0`
- Odoo 20: `odoo-20.0`

If `$ODOO_SOURCE` is set, inspect local framework code for controller dispatch, ORM behavior, decorators, web client actions, assets, tests, and version-specific execution details.

## Tracing Process

1. Locate the entry point with file and line reference.
2. Follow the direct method call chain.
3. Follow inheritance and `super()` calls through parent implementations.
4. Check decorators and implicit triggers: `@api.depends`, `@api.constrains`, `@api.onchange`, inverse methods, `@api.ondelete`, create/write/unlink overrides.
5. Follow XML wiring: buttons, actions, menus, reports, cron records, server actions, and external IDs.
6. Follow frontend wiring when relevant: asset bundle, JS module, registry entry, service call, RPC/ORM call, template event.
7. Record database operations: `search`, `browse`, `read`, `read_group`, `create`, `write`, `unlink`, direct SQL, savepoints, locks.
8. Record side effects: chatter, emails, notifications, activities, reports, attachments, external API calls, queue jobs, bus events.
9. Record security checks: auth mode, ACLs, record rules, groups, domains, company filters, `sudo()` boundaries.
10. Mark exit points and returned actions/data.

## What To Watch For

- N+1 queries from searches, browses, reads, creates, writes, or unlinks inside loops.
- Hidden side effects from computed fields, constraints, onchange methods, mail mixins, activities, and automation.
- Unsafe `sudo()` that bypasses ownership, company, portal, or public-user boundaries.
- Fragile XML linkage through renamed methods, missing external IDs, or version-specific view syntax.
- Transaction hazards from caught database exceptions without savepoints.
- Business rules split across views, onchanges, controllers, and scheduled actions.

## Common Entry Patterns

| Entry Type | Where To Look | Follow Next |
|---|---|---|
| Controller | `controllers/*.py`, `@http.route` | request env, auth, params, model calls |
| Button | XML `<button name="..." type="object">` | model method, overrides, returned action |
| Cron | XML/data `ir.cron` record | model method, batching, side effects |
| Server action | XML/data `ir.actions.server` | model, code block, target records |
| Compute | field `compute="..."`, `@api.depends` | dependencies, store behavior, recompute path |
| Onchange | `@api.onchange`, form view fields | UI-only behavior, persisted equivalent |
| Constraint | `@api.constrains`, SQL constraints | validation path and exception behavior |
| Report | `ir.actions.report`, QWeb template | access, render context, attachments |
| OWL/RPC | `static/src`, services, registries | backend route/model method |

## Output Format

````markdown
## Code Execution Trace

### Entry Point
- Type: Controller / Button / Cron / Server action / Compute / Onchange / Constraint / Report / OWL / Other
- Location: `path/to/file.py:XX`
- Trigger: user action, scheduler, external request, field access, module update, etc.

### Flow
```text
ENTRY: `path/to/file.xml:XX` button/action/route
-> `models/foo.py:42` Foo.action_do_thing()
   -> `models/foo.py:63` Foo._prepare_values()
   -> `models/bar.py:18` Bar.create()
   -> side effect: message_post()
RETURN: `ir.actions.act_window`
```

### Detailed Trace
- `path/to/file.py:XX` method name, decorators, inputs, key branches.
- `path/to/parent.py:XX` parent implementation reached via `super()`.
- `path/to/data.xml:XX` cron/action/report/XML ID involved.

### Database Operations
- `search`: model/domain/count if known.
- `create/write/unlink`: model/count if known.
- Direct SQL/savepoints/locks: state exact location.
- N+1 risk: yes/no/uncertain with reason.

### Security Notes
- Auth mode, ACL/record-rule expectations, groups, company behavior, `sudo()` boundaries.

### Side Effects
- Emails, chatter, activities, attachments, reports, external calls, queue jobs, bus events.

### Uncertain Paths
- State what could not be proven and what file/source would resolve it.
````

## Response Rules

- Always include file/line references for traced code.
- Trace inheritance and `super()` explicitly.
- Do not assume a framework path if local source is available; inspect it.
- Do not run Odoo commands that touch a database or service without user confirmation.
- If the trace reveals review findings, compose with `odoo-code-review`.
