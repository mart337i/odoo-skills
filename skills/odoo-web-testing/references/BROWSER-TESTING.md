# Odoo Browser Testing

## HTTP Controller Tests

Use the version-appropriate Odoo HTTP test base and helpers. Verify the route's public contract:

```python
def test_portal_user_sees_only_owned_record(self):
    response = self.url_open('/my/records/%s' % self.record.id)
    self.assertEqual(response.status_code, 200)
    self.assertIn(self.record.name.encode(), response.content)

    other_response = self.url_open('/my/records/%s' % self.other_record.id)
    self.assertIn(other_response.status_code, (403, 404))
```

Use the actual login and request helpers for the target Odoo version. Do not use an elevated environment in the assertion path when the purpose is access testing.

## Browser and Tour Tests

Use a browser test when the behavior depends on:

- Asset bundle registration or template loading.
- Form validation and submission.
- Navigation, dialogs, menus, or responsive behavior.
- OWL state updates, services, registries, or asynchronous RPC.
- The final rendered result rather than a controller return value.

Prefer stable semantic selectors and user-visible assertions. Avoid selectors tied to generated IDs or unrelated CSS structure.

## Access Matrix

For web-facing data, build a small matrix:

| Actor | Expected result |
| --- | --- |
| Anonymous visitor | Public content only |
| Owning portal user | Own records and allowed actions |
| Other portal user | No access to the record |
| Internal user | Access governed by ACL/rules |
| User in another company | No unintended cross-company access |

## Failure Diagnosis

When a browser test fails, separate these causes:

1. Route/authentication mismatch.
2. ORM access or record-rule failure.
3. Template inheritance or QWeb rendering failure.
4. Missing or misordered assets.
5. Client-side state, service, or asynchronous behavior.

Capture the failing URL, user identity, response status, browser console errors, asset mode, and relevant server log before changing code.

For an OWL-specific diagnosis tree, see
`../../odoo-owl/OWL-DEBUGGING.md`.
