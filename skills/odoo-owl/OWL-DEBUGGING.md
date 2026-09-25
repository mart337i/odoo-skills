# OWL Debugging Decision Tree

Use this reference when an Odoo OWL component, widget, action, or service is
broken. Diagnose the boundary first; do not rewrite reactive code before
checking assets, templates, registries, and browser errors.

## Collect Evidence

- Target Odoo branch and local OWL source.
- Exact URL/action/view and user identity.
- Asset debug mode and browser console output.
- Network requests, response status, and RPC payloads.
- Template name, registry key, manifest bundle, and file path.
- Whether the failure occurs before mount, during render, after an event, or after an async response.

## 1. Nothing Loads

Check:

- The file is declared in the correct manifest bundle.
- The addon is installed and the module was updated when required.
- The asset path is spelled correctly and uses the target bundle name.
- A dependency is declared before an asset operation targets its file.
- The browser is not serving a stale compiled bundle.

Use asset debug mode and inspect the actual loaded files before changing imports.

## 2. Template Not Found

Check:

- `static.template` exactly matches the XML `t-name`.
- The XML template file is in the same effective bundle as the component.
- The XML has loaded before the component renders.
- The template is not accidentally in a backend-only bundle used by a frontend page.
- Inherited templates target an anchor present in the target Odoo version.

## 3. Import Or Registry Failure

Check:

- Import paths match local Odoo source and nearby modules.
- The registry category is correct: actions, fields, services, views, systray, or another target category.
- The key is unique and the registration file is included in the bundle.
- The component/service export matches the import form.
- A legacy module header or modern ES module convention is required by the target.

## 4. Invalid Hook Or Lifecycle Behavior

Check:

- Hooks are called only where the target OWL version permits.
- State is reactive in the actual rendering path.
- Props are not mutated by the child.
- DOM listeners, timers, subscriptions, and external resources are cleaned up.
- Async work cannot write into a destroyed component.
- Error boundaries or explicit event-handler recovery are used where needed.

Do not apply OWL 3 signals/plugins or OWL 2 hooks solely from the Odoo version
number. Confirm the local OWL implementation first.

## 5. RPC Or Data Failure

Separate client and server causes:

- Inspect the network request and response first.
- Confirm model, method, arguments, domain, and field names.
- Check user access, record rules, company context, and controller authentication.
- Check whether the component handles loading, empty, error, and retry states.
- Keep authorization in the server model/controller, never only in the component.

## 6. CSS Or Rendering Failure

Check:

- SCSS is in the correct bundle and compiled without errors.
- Selectors use the module/component namespace.
- The element is actually present and not hidden by a conditional template branch.
- Responsive behavior, RTL, focus states, and disabled states remain usable.
- External overlay/popover components render outside the component's immediate DOM.

## Verification Order

1. Reproduce with asset debug mode.
2. Confirm network and console evidence.
3. Confirm manifest bundle and template loading.
4. Confirm registry/import wiring.
5. Confirm server response and access behavior.
6. Confirm state/lifecycle cleanup.
7. Add a regression test at the smallest seam that reproduces the failure.
