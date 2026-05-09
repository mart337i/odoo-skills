# OWL 3 Templates

OWL templates use QWeb-style XML syntax with OWL-specific directives. Verify local OWL/Odoo version before applying OWL 3-specific syntax.

## Template Basics

- Directives are XML attributes prefixed with `t-`.
- Use `<t>` as a placeholder when you need a directive without rendering an extra element.
- Expressions are JavaScript expressions, not statements.
- Component properties and methods are accessed with `this` in OWL 3 templates.

Valid:

```xml
<p t-if="this.user.birthday === this.today()">Happy birthday!</p>
```

Invalid:

```xml
<p t-if="console.log(1)">Not valid</p>
```

## Core Directives

- `t-out`: output escaped data.
- `t-set`, `t-value`: set scoped variables.
- `t-if`, `t-elif`, `t-else`: conditionals.
- `t-foreach`, `t-as`, `t-key`: loops and reconciliation keys.
- `t-att`, `t-att-*`, `t-attf-*`: dynamic attributes.
- `t-call`: render sub templates.
- `t-debug`, `t-log`: debugging helpers.

OWL-specific directives:

- `t-component`, `t-props`: dynamic components and props.
- `t-ref`: DOM/component refs.
- `t-on-*`: event handling.
- `t-call-slot`, `t-set-slot`, `t-slot-scope`: slots.
- `t-model`: form bindings.
- `t-tag`: dynamic tag names.
- `t-custom-*`: custom directives.

## Output And Safe HTML

`t-out` escapes by default. Only use `markup` when the value is intentionally safe HTML.

```js
import { markup } from "@odoo/owl";

const value = markup`<b>${userInput}</b>`;
```

Treat `markup` as a security boundary. Do not use it to silence escaping unless the source is trusted or properly escaped.

## Loops And Keys

Always use stable `t-key` values for dynamic lists. Keys should be unique strings or numbers, not objects.

```xml
<li t-foreach="this.items" t-as="item" t-key="item.id">
  <t t-out="item.label"/>
</li>
```

Avoid index keys for reorderable lists because they can lose DOM state such as selection, input value, scroll position, or animation state.

For `Set` or other iterables, convert to an array in templates if the local OWL version requires it:

```xml
<t t-foreach="[...this.items]" t-as="item" t-key="item.id">
  ...
</t>
```

## Events

Use `t-on-event="this.method"` or an expression returning a function.

```xml
<button t-on-click="this.save">Save</button>
<button t-on-click="() => this.increment(3)">Add 3</button>
```

Modifiers:

- `.stop`: stop propagation.
- `.prevent`: prevent default.
- `.self`: only when target is the element itself.
- `.capture`: capture mode.
- `.synthetic`: document-level synthetic event for large lists.
- `.passive`: passive listener, useful for scroll/touch events.

Do not combine `.passive` and `.prevent`; they contradict each other.

Modifier order can matter. `t-on-click.prevent.self` prevents all clicks, while `t-on-click.self.prevent` only prevents clicks on the element itself.

## Dynamic Attributes

Use `t-att-class` object syntax for conditional classes:

```xml
<div class="base" t-att-class="{ active: this.isActive(), disabled: this.disabled() }"/>
```

Use `t-att-style` object syntax for dynamic inline styles when needed, but prefer classes for reusable styling.

## Inline Vs External Templates

Inline `xml` templates are convenient for examples and small standalone code. In Odoo addons, prefer external XML templates when translation extraction, inheritance, or asset tooling matters.

## SVG

OWL uses heuristics for SVG namespaces. Component or sub-template roots used inside SVG should generally be `svg`, `g`, or `path`.

## Restrictions

OWL templates forbid tags and attributes starting with `block-` to avoid internal name collisions.
