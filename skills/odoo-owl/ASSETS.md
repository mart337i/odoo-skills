# OWL Asset Integration

Use this reference when an OWL component, field widget, service, template, or
style is not loading correctly in an Odoo addon. Verify the target Odoo source
and existing module conventions before applying an example.

## First Check

Inspect `__manifest__.py` and answer:

- Which bundle should load the code: backend, frontend, POS, tests, or a local bundle?
- Are JavaScript, XML templates, and SCSS all included?
- Does the addon depend on the module that declares an asset being replaced or positioned?
- Does the target version use the same module/import conventions as nearby code?

## Typical Manifest Declaration

```python
'assets': {
    'web.assets_backend': [
        'my_module/static/src/components/my_component/my_component.js',
        'my_module/static/src/components/my_component/my_component.xml',
        'my_module/static/src/components/my_component/my_component.scss',
    ],
    'web.assets_unit_tests': [
        'my_module/static/tests/**/*.js',
    ],
},
```

Use `web.assets_frontend` for public website behavior and the target version's
documented test bundle for JavaScript tests. Do not add a legacy QWeb bundle
just because a pattern from another Odoo version uses one.

## Asset Operations

Manifest asset lists support operations whose exact syntax must be checked
against the target version:

```python
'web.assets_backend': [
    'my_module/static/src/**/*.js',
    ('prepend', 'my_module/static/src/js/bootstrap.js'),
    ('after', 'web/static/src/js/target.js',
             'my_module/static/src/js/extension.js'),
    ('replace', 'web/static/src/js/old.js',
               'my_module/static/src/js/new.js'),
    ('remove', 'web/static/src/js/unused.js'),
],
```

The target file for `before`, `after`, `replace`, and `remove` must already be
declared. A module changing another module's asset list normally needs a
dependency on that module.

## Loading Order

- Put a dependency before a glob that would otherwise include it later.
- Keep setup/polyfill files before code that imports or uses them.
- Keep XML templates in the same effective bundle as the component that uses them.
- Avoid broad globs when they make ordering or bundle size unclear.
- Use lazy loading for large libraries that are needed only by a specific flow.

## Lazy Loading

Use the target version's asset helper or hook for on-demand libraries. Keep the
loader in a small module and make failure behavior visible to the component:

```javascript
import { loadJS } from "@web/core/assets";

export async function loadChartLibrary() {
    await loadJS("/my_module/static/lib/chart/chart.min.js");
    return window.Chart;
}
```

Confirm the helper path in local Odoo source. Do not assume that a helper's
name or import path is stable across major versions.

## Debugging Assets

1. Open the page with the target version's asset debug mode.
2. Check the network response for the JavaScript and XML files.
3. Confirm the manifest path and bundle name exactly match the file paths.
4. Check the browser console for import, template, and registry errors.
5. Check whether an inherited bundle operation ran before its target existed.
6. Upgrade or reload the module only after confirming the local command and database.

## Review Checklist

- [ ] Correct bundle for the execution context.
- [ ] JS, XML, SCSS, and tests are declared where needed.
- [ ] Asset operations target files that are already declared.
- [ ] Dependencies match imported modules and asset overrides.
- [ ] Templates are loaded before component registration/use.
- [ ] Large libraries are lazy-loaded where practical.
- [ ] Debug-assets and cache behavior have been checked.
