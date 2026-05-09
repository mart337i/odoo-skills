# OWL 3 State And Lifecycle

Verify target OWL version before applying OWL 3 APIs.

## Reactivity Model

OWL 3 reactive values are not tied to components. They can live in components, plugins, or plain modules. Dependency tracking happens when reactive values are read during rendering, effects, or computed values.

Updates are batched in microtasks, so several synchronous writes usually produce one rerender/effect run.

## Signals

Use `signal(value)` for scalar state.

```js
count = signal(0);

increment() {
  this.count.set(this.count() + 1);
}
```

Read with `this.count()`. Write with `this.count.set(value)`.

Setting an identical value is a no-op.

## Collection Signals

Use collection signals when mutating arrays/objects/maps/sets in place:

- `signal.Array([])`
- `signal.Object({})`
- `signal.Map(new Map())`
- `signal.Set(new Set())`

They detect shallow mutations. Deep nested mutations are not detected. Prefer replacing nested objects or modeling nested state explicitly.

Use `signal.trigger(signalValue)` only when collection signals are not appropriate.

## Computed Values

Use `computed()` for derived state.

```js
total = computed(() => this.lines().reduce((sum, line) => sum + line.amount, 0));
```

Computed dependencies are tracked dynamically from the last evaluation. Use computed values instead of effects when the goal is derived state.

## Proxy

Use `proxy()` for reactive object graphs.

```js
state = proxy({ text: "", items: [] });
```

`proxy` is not a hook and can be called outside components. Nested objects are recursively proxied.

## Effects

Use `useEffect()` in components/plugins so cleanup is automatic. Use raw `effect()` only for non-component lifetime and remember to dispose it.

Effects run immediately and rerun when reactive values read during execution change. If an effect returns a function, that cleanup runs before the next rerun.

Avoid creating nested effects accidentally. If an independent inner effect is needed, use `untrack()` and keep the disposer.

## Hook Rule

Hooks must be called in `setup()` or class fields. Do not call hooks later in lifecycle methods or event handlers.

Allowed:

```js
class C extends Component {
  state = proxy({ value: 0 });

  setup() {
    useEffect(() => this.state.value);
  }
}
```

Not allowed:

```js
async willStart() {
  this.state = proxy({ value: 0 });
}
```

## Lifecycle Hooks

- `onWillStart`: async work before first render. Keep fast; multiple callbacks run in parallel.
- `onMounted`: after component is rendered and attached to DOM.
- `onWillPatch`: before DOM patch; read DOM state only, do not update reactive state.
- `onPatched`: after DOM patch; DOM interaction allowed, but avoid update loops.
- `onWillUnmount`: before removing a mounted component from DOM.
- `onWillDestroy`: always called before destroy, even if never mounted.
- `onError`: catches descendant rendering/lifecycle errors.

Use `onWillDestroy` for cleanup that must always run. Use `onWillUnmount` for DOM-mounted cleanup that only exists after mount.

## Async Cancellation

Use the provided `abortSignal` in `onWillStart` and other async hooks when possible.

```js
onWillStart(async ({ abortSignal }) => {
  const response = await fetch("/api/data", { signal: abortSignal });
  this.data = await response.json();
});
```

For APIs that do not accept `AbortSignal`, use `scope.until(promise)` or `abortSignal.throwIfAborted()` between awaits.

Never write async results into a destroyed component. Scope cancellation exists to prevent stale writes.

## Refs

In OWL 3, refs are signal-based.

```js
inputRef = signal(null);

focusInput() {
  this.inputRef()?.focus();
}
```

```xml
<input t-ref="this.inputRef"/>
```

For refs inside loops, use `Resource` instead of one signal.

## Error Handling

Uncaught render/lifecycle errors destroy the app because OWL cannot guarantee tree consistency. Use `onError` or `ErrorBoundary` around risky subtrees.

Errors from event handlers are not managed by `onError`; application code must handle recovery.

Fallback UI must be safe and should not throw, or it can create an error loop.
