---
name: odoo-grill-me
description: Odoo-specific grilling session that stress-tests addon plans, migrations, model/view/security designs, OWL frontend changes, and verification strategy. Use when the user wants to be grilled on an Odoo plan/design, asks to stress-test an addon architecture, or mentions grill me in an Odoo context.
---

# Odoo Grill Me

Interview the user relentlessly about every aspect of an Odoo plan until there is shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one by one.

Ask exactly one question at a time. For each question, include your recommended answer.

If a question can be answered by exploring the codebase, inspect the codebase instead of asking.

## First Move

Before asking, gather discoverable facts:

- Odoo version and edition if visible from branch/docs/config.
- Addon/module names and manifests.
- Existing models, views, security, data, controllers, assets, tests, and docs relevant to the plan.
- `$ODOO_SOURCE`, `$ODOO_TOOL_README`, and `$ODOO_BASE_COMMAND` context when available.

Then ask the highest-leverage unresolved question.

## Question Branches

Use only the branches relevant to the plan.

### Product And Workflow

- What user workflow is being changed?
- Which users/groups need it?
- What should happen in draft, confirmed, cancelled, archived, or error states?
- What business invariant must never be violated?

### Module Boundary

- Which addon owns the concept?
- Is this a new addon, extension of an existing addon, or configuration/data-only change?
- Which manifest dependencies are real, and which would be convenience dependencies?
- Does the plan couple unrelated business domains?

### Model Design

- Is this a new model, `_inherit`, or `_inherits`?
- What is the lifecycle of the record?
- Which fields are stored, computed, related, required, tracked, or company-dependent?
- Which constraints belong in Python, SQL, views, or onchange?
- Which methods are stable seams for future extension?

### Security And Access

- Which groups can read, create, write, and unlink?
- Are record rules needed, and are they multi-company safe?
- Is any `sudo()` required? If yes, why is access elevation correct?
- Are portal/public users or controllers exposed?
- What data could leak across companies, websites, portals, or users?

### Views, Actions, Menus, Reports, Wizards

- Where does the user enter and complete the workflow?
- Are inherited views targeting stable anchors?
- Do actions, menus, and reports load after their dependencies?
- Is a wizard transient state, or is it hiding a missing domain model?
- Does the UI rely on onchange-only logic for persisted behavior?

### Data And XML

- Which XML IDs must stay stable?
- Is data production data, demo data, configuration, or noupdate reference data?
- What is the manifest load order?
- What happens on module update?

### Controllers And Integrations

- What is the trust boundary?
- Which `auth` and CSRF settings are correct?
- What input validation and access checks happen before ORM access?
- Are external calls retried, timed out, logged, or mocked in tests?

### OWL And Assets

- Which Odoo/OWL version is targeted?
- Which asset bundle loads the component/template/style?
- Is state owned by the parent, component, plugin/service, or backend model?
- Are props readonly and explicit?
- Are listeners, intervals, refs, and async work cleaned up?

### Migration

- Is this a normal feature change or an OCA module migration?
- What are the source and target versions?
- Which intermediate version checklists apply?
- Are compatibility changes separated from behavior changes?

### Testing And Verification

- What is the smallest test that proves the Odoo behavior seam?
- Are access rules tested with realistic users/groups/companies?
- Does the plan need an ORM test, module update, shell snippet, tour, report render, or controller request?
- Which local command should be used, and has the user confirmed it?

## Stopping Rule

Stop grilling when:

- the module boundary is clear;
- model, UI, security, data, and verification choices are coherent;
- remaining risks are explicit and accepted;
- the next step is implementable without guessing.

Then summarize the agreed design and list the accepted risks.
