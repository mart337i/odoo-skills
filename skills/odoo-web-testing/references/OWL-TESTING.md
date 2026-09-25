# OWL Testing Patterns

Use this reference for Odoo JavaScript and OWL tests. Test utility imports,
service mocks, registry helpers, and asset bundles are version-sensitive; copy
the shape of an existing test in the target Odoo source first.

## Choose The Seam

- Component rendering and state transitions: JavaScript component test.
- Service, registry, or field-widget integration: JavaScript integration test.
- Asset/template loading and navigation: browser tour or `HttpCase` flow.
- Business rules and access rights: ORM/security test, not a frontend mock.
- Public or portal behavior: combine browser/HTTP testing with the access matrix.

## Component Test Shape

```javascript
QUnit.module("my_module.MyComponent", (hooks) => {
    let target;

    hooks.beforeEach(() => {
        target = getFixture();
    });

    QUnit.test("renders an empty state", async (assert) => {
        await mount(MyComponent, target, {
            props: { records: [] },
        });

        assert.strictEqual(
            target.querySelector(".o_my_empty_state").textContent.trim(),
            "No records",
        );
    });
});
```

Resolve `getFixture`, `mount`, and assertion helpers from the target version's
existing tests. Do not assume a helper path from another major version.

## Service And ORM Mocks

Mock at the service boundary, not deep inside the component implementation.
Assert both the returned UI state and the call arguments:

```javascript
const orm = {
    searchRead: async (model, domain, fields) => {
        assert.strictEqual(model, "my.model");
        assert.deepEqual(domain, [["active", "=", true]]);
        return [{ id: 1, name: "Record" }];
    },
};
```

Use the target test environment's service registry/mocking helper to inject the
mock. If no helper exists, keep the adapter small and restore it in teardown.

## State Cases

For data-loading components, cover:

- Loading state before the request resolves.
- Successful data state.
- Empty result state.
- Request failure and visible recovery action.
- Retry behavior.
- Response arriving after unmount or cancellation.

For field widgets, also cover readonly mode, empty values, invalid options, and
the exact `record.update()` payload.

## Registry And Asset Tests

- Verify the component, service, action, or field widget is registered under the expected key.
- Verify the XML template is included in the bundle used by the test.
- Verify the manifest asset path matches the actual file path.
- Use a browser/tour test when the failure could only be detected after real asset loading.

## Browser Boundary

Use stable semantic selectors and user-visible assertions. Avoid generated IDs,
implementation-only CSS classes, arbitrary sleeps, and assertions on minified
asset filenames. For portal/public flows, test anonymous, owning, non-owning,
internal, and cross-company users as applicable.

## Test Review Checklist

- [ ] Test helpers match the target Odoo version.
- [ ] Services and RPC are mocked at the boundary.
- [ ] Async work is awaited and cancellation is covered.
- [ ] Loading, success, empty, and error states are covered.
- [ ] Registry and template loading are verified.
- [ ] Browser tests cover the real asset/template seam where needed.
- [ ] Tests assert behavior rather than DOM implementation details.
