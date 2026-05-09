# OWL 3 Core

Use this as a compact map of OWL 3 core concepts. Verify local OWL version before applying it in Odoo projects.

## What OWL Is

OWL is a class-based UI framework built by Odoo. OWL 3 uses:

- signal-based reactivity
- class components
- XML/QWeb-style templates
- async rendering
- plugins for shared state/services
- no mandatory toolchain for native ES module usage

## Main Exports

Core:

- `App`: OWL application root manager.
- `Component`: base class for components.
- `Suspense`: fallback rendering while descendant async `onWillStart` is pending.
- `ErrorBoundary`: fallback rendering when descendants throw.
- `Portal`: render content elsewhere in the DOM.
- `mount`: mount a root component.
- `xml`: define inline templates.
- `props`: declare and validate component props.
- `status`: read component status.

Reactivity:

- `signal`: reactive value.
- `computed`: lazily-evaluated derived value.
- `proxy`: reactive object/array/map/set proxy.
- `effect`: side effect that reruns when dependencies change.
- `markRaw`, `toRaw`, `untrack`: reactivity escape hatches.

Plugins and lifetime:

- `Plugin`: base class for shared state/logic.
- `plugin`: import a plugin dependency.
- `providePlugins`: provide plugins to a subtree.
- `Scope`: lifetime handle for components and plugins.

Collections and utilities:

- `Registry`: ordered reactive key-value collection.
- `Resource`: ordered reactive set-like collection.
- `EventBus`, `markup`, `batched`, `whenReady`.

## Components

Components subclass `Component` and usually define:

- `static template`: template name or inline `xml` template.
- `static components`: child component registry used by the template.
- reactive fields such as `signal`, `computed`, or `proxy`.
- `setup()`: hook registration and initialization.
- methods called by templates/events.

Example:

```js
import { Component, signal, xml } from "@odoo/owl";

class Counter extends Component {
  static template = xml`
    <button t-on-click="this.increment">
      <t t-out="this.count()"/>
    </button>`;

  count = signal(0);

  increment() {
    this.count.set(this.count() + 1);
  }
}
```

## Composition

Register child components with `static components` and use capitalized tags in templates.

```js
class Parent extends Component {
  static template = xml`<Child value="1"/>`;
  static components = { Child };
}
```

Use `t-component` only for dynamic component classes, and keep that dynamic choice explicit and testable.

## Props

Props are owned by the parent and should be treated as readonly by the child. In OWL 3, a component explicitly imports props using `props()` or `prop()`.

Use `props()` when values may change or multiple props are needed. Use `prop()` for a small number of identity-stable props such as models, event buses, or signals.

Typed props are validated in dev mode:

```js
import { Component, props, types as t } from "@odoo/owl";

class ProductList extends Component {
  props = props({
    items: t.array(t.object({ id: t.number(), label: t.string() })),
    "onSelect?": t.function(),
  });
}
```

If a component validates props and uses slots, include `slots?` when it needs to consume or forward slots.

## Plugins Replace Env In OWL 3

OWL 3 removes OWL 2's `env`. Shared state/services should be modeled as plugins.

```js
class Notifications extends Plugin {
  items = signal.Array([]);

  add(message) {
    this.items().push({ id: Date.now(), message });
  }
}

class Header extends Component {
  notifications = plugin(Notifications);
}
```

Use `providePlugins()` for subtree-scoped services. Use app-level plugins for global state.
