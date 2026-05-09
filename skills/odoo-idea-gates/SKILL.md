---
name: odoo-idea-gates
description: Odoo-aware idea validation for deciding whether a module, addon, app, or product should exist before implementation. Use when the user wants to validate an Odoo module idea, check whether a solution already exists, assess usefulness or buyer demand, narrow a wedge, or decide whether an addon is worth building.
---

# Odoo Idea Gates

Validate the idea before designing or building it. Kill, narrow, or strengthen weak Odoo module/product ideas using evidence.

Use this skill before implementing a new Odoo module, OCA addon, marketplace app, internal tool, or SaaS/product idea connected to an Odoo workflow.

## First Move

Gather discoverable facts before asking:

- Search the current repo for related addons, models, views, settings, wizards, server actions, cron jobs, integrations, and docs.
- Check whether the idea is already solved by Odoo standard, Enterprise, Studio/configuration, an OCA addon, an Odoo Apps module, or an existing custom addon.
- Detect the Odoo version, edition, industry, and target workflow if visible from branch names, manifests, docs, or config.
- If `$ODOO_SOURCE` is set, inspect framework features before assuming a custom addon is needed.

If evidence is missing, ask exactly one high-leverage question at a time. Include your recommended answer or likely disqualifier.

## Gate Workflow

Apply gates in order. Stop early when an idea clearly fails unless the user asks to continue.

| Gate | Pass Test | Kill Signal |
|---|---|---|
| 1. Insider advantage | User has firsthand workflow experience or direct access to users. | Idea came from marketplace browsing, generic interest, or guessed workflow details. |
| 2. Existing solution search | Clear gap remains after checking Odoo standard, Enterprise, Studio/configuration, OCA, Odoo Apps, custom addons, and adjacent tools. | The solution already exists and can be used, configured, bought, or contributed to. |
| 3. Existing spend | Buyers already spend time, money, or consultant budget on adjacent pain. | No one pays for anything near the problem; demand must be created. |
| 4. Reachable buyers | User can name 50 likely buyers/users and reach five for discovery within a week. | Buyer is abstract or distribution depends on marketplace luck. |
| 5. Wedge narrowness | v1 is narrow by Odoo version, edition, industry, model/workflow, role, company size, and workaround. | It serves many industries, roles, or workflows; example bad wedge: `project management for Odoo users`. |
| 6. Odoo fit | Smallest correct shape is clear: config, Studio, extension, OCA contribution, server action, wizard/report tweak, or new addon. | It fights standard Odoo workflow, needs fragile view hacks, broad `sudo()`, or unnecessary new domain models. |
| 7. Build/maintenance cost | Security, multi-company, migrations, integrations, reporting/accounting/stock risk, and testability are proportionate. | Small pain requires deep risky changes or upgrades would consume the business. |
| 8. Value equation | `(Dream Outcome x Perceived Likelihood) / (Time Delay x Effort)` is strong and first value is under 30 minutes. | Buyer must reorganize Odoo, clean lots of data, or rely on consultant-led rollout before seeing value. |

Good wedge format: `Resource utilization tracking for 5-15 person web dev agencies using Odoo Project, Timesheets, and Harvest.`

## Question Prompts

Use only the prompts needed to unblock the next gate:

- What exact Odoo workflow have you personally used, sold, supported, implemented, or watched users struggle through?
- Which role feels the pain: accountant, salesperson, warehouse operator, project manager, consultant, portal user, partner, or admin?
- Can you name three adjacent spends: Enterprise seats, Studio, paid apps, OCA maintenance, custom development, partner retainers, admin time, spreadsheets, external SaaS, or middleware?
- Can you name 50 target companies, partners, implementers, or Odoo users within a week?
- What one dimension can be 10x better than existing Odoo/OCA/App Store alternatives?
- Could configuration, Studio, an existing addon extension, or OCA contribution solve this without a new module?
- What data must already exist in Odoo, and can a real user see value within 30 minutes?

## Output Format

End with one verdict:

- `kill`: do not build; explain the fatal gate.
- `narrow`: preserve the insight but shrink the wedge.
- `research`: evidence is missing; list the smallest validation step.
- `proceed`: define the first addon boundary and validation plan.

Include:

- Gate results with pass/fail/unknown.
- Existing alternatives found or still needing search.
- The narrowest viable wedge.
- The first validation action before implementation.
- The main Odoo technical risk if the idea proceeds.

## Biases

- Prefer boring, specific Odoo workflow pain over broad platform ideas.
- Prefer capturing existing demand over creating demand.
- Prefer configuration, contribution, or extension over a new addon when sufficient.
- Prefer one 10x improvement for one role over a broad feature set.
- Treat missing buyer access as a product risk, not a marketing detail.
