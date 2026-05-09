# OWL 1/2 To OWL 3 Migration

OWL 3 migration guidance is version-sensitive. The OWL 2 to OWL 3 guide is marked draft upstream and was written primarily for Odoo's own codebase. Always verify local OWL/Odoo source and existing compatibility layers before editing.

## Migration Workflow

1. Detect current OWL version and target OWL version.
2. Inspect existing imports, compatibility helpers, and local migration scripts.
3. Identify component patterns: state, props, env/services, refs, model bindings, lifecycle hooks, template expressions.
4. Convert one vertical slice at a time.
5. Prefer semantic conversions over blind search/replace when state or lifecycle changes are involved.
6. Run the repo's frontend tests/assets build/module update after user confirmation where needed.

## Breaking Changes Checklist

State and reactivity:

- `useState` removed; replace with `proxy` or `signal` depending on state shape.
- `reactive` removed; replace simple one-argument usage with `proxy`.
- `reactive(value, callback)` usually becomes `proxy` plus `useEffect`, `effect`, or better, `computed`.
- `useEffect` no longer takes a dependency function/array; dependencies are tracked by reactive reads.
- Derived state should usually become `computed` instead of `useEffect`.

Props:

- Built-in `this.props` is removed; call `props()` or `prop()` explicitly.
- Static `props` and `defaultProps` are ignored; convert to `props(schema, defaults)`.
- Props used for derived values often need to be signals for correct subscription.
- Props are readonly from the child perspective.

Environment and services:

- `this.env` is removed.
- Replace `env`, `useEnv`, `useSubEnv`, and services with plugins where possible.
- `providePlugins()` replaces subtree environment extension.
- `plugin(SomePlugin)` replaces reading shared service/state from env.

Templates:

- Rendering context generally requires `this.` for component properties/methods.
- `t-esc` removed; use `t-out`.
- Be careful when replacing `t-esc` in code that may run temporarily on OWL 2 and output objects.
- `t-call` has stricter usage; verify local target syntax.
- `t-portal` removed; use supported portal APIs/components for target version.

Refs and form bindings:

- `t-ref` takes a signal or resource.
- Replace `useRef("name")` with `signal(null)` and `t-ref="this.someRef"`.
- Read refs with `this.someRef()`.
- Use `Resource` for refs inside loops.
- `t-model` takes a signal in OWL 3; adapt readers/writers accordingly.

Lifecycle:

- `onWillUpdateProps` removed; use `computed`, `useEffect`, or async patterns with cancellation.
- `onWillRender` removed; use `computed` for precomputed values or a proper mounted/effect hook for side effects.
- `onRendered` removed; use `onMounted` or `onPatched` depending on intent.
- `this.render` removed; use reactive state instead of forced rendering.
- `useExternalListener` renamed/replaced by `useListener`, with changed semantics.
- `useComponent` removed; return explicit hook state or use props/plugins directly.

## Conversion Examples

`useState` to `proxy`:

```js
// OWL 2
this.state = useState({ value: 1 });

// OWL 3
this.state = proxy({ value: 1 });
```

Forced render to signal:

```js
// OWL 2
value = 1;
increment() {
  this.value++;
  this.render();
}

// OWL 3
value = signal(1);
increment() {
  this.value.set(this.value() + 1);
}
```

Static props to `props()`:

```js
props = props(
  {
    name: t.string(),
    "visible?": t.boolean(),
  },
  { visible: true }
);
```

Ref migration:

```js
// OWL 3
divRef = signal(null);

setup() {
  onMounted(() => {
    console.log(this.divRef());
  });
}
```

```xml
<div t-ref="this.divRef"/>
```

## Migration Review Risks

High-risk conversions:

- env/service replacement, because it changes shared-state architecture.
- `reactive(value, callback)` with side effects or derived state.
- async `onWillUpdateProps` replacements without cancellation.
- forced render removal when the underlying state is not reactive.
- refs and model bindings in loops or external-library integrations.
- event handlers relying on old binding behavior.

Low-risk conversions:

- `t-esc` to `t-out` when target is OWL 3 only.
- `useState` to `proxy` for simple object state.
- `useExternalListener` to `useListener` when target signature is verified.

## Output For Migration Reviews

When reviewing or planning a migration, return:

- Current OWL version evidence.
- Target OWL version evidence.
- Files/patterns affected.
- Mechanical changes possible.
- Manual changes requiring design judgement.
- Verification seams: tests, assets, module update, browser checks.
