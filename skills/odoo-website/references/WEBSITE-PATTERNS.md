# Odoo Website Patterns

## Controller Discovery

Before adding a route, inspect the nearest standard controller and answer:

1. Which module owns the business flow?
2. Is the route public, portal, or internal?
3. Should it use `http` or `json` request handling?
4. Which recordset and ownership checks protect the requested resource?
5. Which template, asset bundle, and translation conventions already exist?

Prefer extending a standard controller or calling a model method over duplicating business logic in a controller.

## Public Page

```python
from odoo import http
from odoo.http import request


class CatalogController(http.Controller):
    @http.route('/catalog', type='http', auth='public', website=True)
    def catalog(self, **kwargs):
        products = request.env['product.template'].search(
            [('website_published', '=', True)],
            limit=20,
        )
        return request.render('my_module.catalog_page', {
            'products': products,
        })
```

Public routes must only expose records intentionally made public. Do not accept a record ID and call `sudo()` without an explicit publication and ownership policy.

## Portal Record Access

Use the portal user's access and ownership model as the source of truth. A typical flow is:

```python
record = request.env['my.record'].browse(record_id).exists()
if not record or not record._can_be_seen_by_portal_user():
    raise werkzeug.exceptions.NotFound()
```

The exact helper, token flow, and exception should follow the detected Odoo version and nearby standard portal controllers. Do not rely on an ID being difficult to guess.

## Website Assets

Declare only the files needed by the page or client:

```python
'assets': {
    'web.assets_frontend': [
        'my_module/static/src/js/catalog.js',
        'my_module/static/src/scss/catalog.scss',
        'my_module/static/src/xml/catalog.xml',
    ],
},
```

Check bundle choice, dependency order, template loading, debug-assets behavior, and cache invalidation before debugging frontend code.

## Review Checklist

- Route auth and request type match the user flow.
- Inputs are normalized and validated before ORM calls.
- Record access is checked before rendering or mutation.
- Errors do not reveal whether another user's record exists.
- Templates escape user-controlled values and include translations.
- Forms work on mobile and preserve CSRF protection where applicable.
- Controller, access, template, and asset tests cover the behavior seam.
