# Odoo 20 OWL Guide

Guide for building OWL (Owl Web Library) components in Odoo 20.

## Table of Contents

- [OWL Overview](#owl-overview)
- [Component Structure](#component-structure)
- [Hooks](#hooks)
- [Services](#services)
- [Public Website Interactions](#public-website-interactions)
- [State Management](#state-management)
- [QWeb Templates](#qweb-templates)
- [Translations](#translations)

---

## OWL Overview

OWL is a JavaScript framework for building web UI components in Odoo.

Odoo 20 uses OWL 3. Code copied from Odoo 16-19 often needs active migration before it can run correctly.

### Odoo 20 Web Client Notes

- Font Awesome icons have been replaced by Material Symbols in the Odoo 20 UI. Before adding or migrating an icon, inspect the target component/template for the supported icon syntax instead of copying older `fa-*` classes.
- Offline mode can create, edit, archive, unarchive, and delete records, and can reuse previous searches. Custom components that write data should handle RPC failure/retry states and avoid assuming a write either succeeded immediately or failed permanently.
- Push notifications no longer use Firebase; they use Odoo's internal push tooling. Do not add Firebase dependencies for 20.0 notifications without verifying the target app still requires them.
- Many2one fields have improved backend synchronization and large-list behavior. Custom field widgets should pass dynamic `domain` and `context` through the standard props and verify the saved backend value after onchange/save.

### Key Concepts

| Concept       | Description                |
| ------------- | -------------------------- |
| **Component** | Reusable UI building block |
| **State**     | Reactive data              |
| **Props**     | Component properties       |
| **Hooks**     | Lifecycle functions        |
| **Template**  | QWeb template              |

---

## Component Structure

### Basic Component

```javascript
import { Component, t, useProps } from "@odoo/owl";

export class MyComponent extends Component {
  static template = "my_module.MyComponent";
  props = useProps({
    value: t.string().optional(),
  });

  setup() {
    // Component setup
  }
}
```

### Register Component

```javascript
import { registry } from "@web/core/registry";

registry.category("actions").add("my_component", MyComponent);
```

### Use in View

```xml
<widget name="my_component" options="{'value': 'Hello'}"/>
```

---

## Hooks

OWL 3 rejects static `props` and `defaultProps` through Odoo's compatibility layer. Declare props with `useProps(...)`. OWL 2-style `useEffect(effect, deps)` is also invalid; migrate to OWL 3 signal-based `useEffect` or use `useLayoutEffect` temporarily when following Odoo's compatibility-layer migration path.

### Setup Hook

Called when component is created:

```javascript
setup() {
    // Initialize state
    this.state = useState({ count: 0 });

    // Call services
    this.rpc = useService("rpc");
    this.orm = useService("orm");
    this.action = useService("action");
}
```

### Lifecycle Hooks

| Hook              | When                  |
| ----------------- | --------------------- |
| `setup()`         | Component creation    |
| `onWillStart()`   | Before render (async) |
| `onMounted()`     | After render          |
| `onWillUnmount()` | Before destroy        |
| `onWillPatch()`   | Before update         |
| `onPatched()`     | After update          |

### Example

```javascript
setup() {
    onWillStart(this.onWillStart);
    onMounted(this.onMounted);
    onWillUnmount(this.onWillUnmount);
}

async onWillStart() {
    // Load data before render
}

onMounted() {
    // After render
}

onWillUnmount() {
    // Cleanup
}
```

---

## Services

Odoo 20 is moving frontend infrastructure from services toward OWL plugins and resources. `useService("orm")`, `useService("action")`, and other compatibility service calls still exist for many surfaces, but imports ending in `_service.js` are migration-risk patterns. For converted code, prefer the plugin-aware API exposed by the target source.

Direct reads of `env.debug` and `env.isSmall` are not Odoo 20-safe. Use `usePlugin(DebugModePlugin)` or global `odoo.debug` for debug state, and `useService("ui").isSmall`, `env.services.ui.isSmall`, or `UIPlugin` signals for viewport state. Do not rely on `Component.env` being initialized globally by the webclient.

### Common Services

| Service        | Description         |
| -------------- | ------------------- |
| `orm`          | Database operations |
| `rpc`          | RPC calls           |
| `action`       | Execute actions     |
| `dialog`       | Show dialogs        |
| `notification` | Show notifications  |
| `router`       | Navigation          |
| `user`         | Current user        |
| `company`      | Current company     |

### Use Service

```javascript
setup() {
    this.orm = useService("orm");
    this.rpc = useService("rpc");
    this.action = useService("action");
    this.dialog = useService("dialog");
    this.notification = useService("notification");
}
```

### ORM Service

```javascript
// Search
const records = await this.orm.search("my.model", [["active", "=", true]]);

// Read
const data = await this.orm.read("my.model", ids, ["name", "value"]);

// Create
const id = await this.orm.create("my.model", { name: "Test" });

// Write
await this.orm.write("my.model", [id], { name: "Updated" });

// Unlink
await this.orm.unlink("my.model", [id]);
```

### RPC Service

```javascript
// Call controller
const result = await this.rpc("/my/controller", { param: "value" });
```

### Action Service

```javascript
// Do action
await this.action.doAction({
  type: "ir.actions.act_window",
  res_model: "my.model",
  views: [
    [false, "list"],
    [false, "form"],
  ],
});
```

### Dialog Service

```javascript
// Add dialog
this.dialog.add(MyDialog, {
    title: "My Dialog",
    confirm: () => {...},
});
```

### Notification Service

```javascript
// Show notification
this.notification.notify({
  message: "Success!",
  type: "success",
});
```

---

## Public Website Interactions

Odoo 20 public website JavaScript no longer targets the old public-widget stack. Migrate code using `@web/legacy/js/public/public_widget`, `@web/legacy/js/public/minimal_dom`, `publicWidget.Widget.extend(...)`, or `root.widget` to `Interaction` classes.

```javascript
import { registry } from "@web/core/registry";
import { Interaction } from "@web/public/interaction";

export class MyPublicInteraction extends Interaction {
    static selector = ".my_public_selector";
    dynamicContent = {
        _root: { "t-on-click": this.onClick },
    };

    onClick() {
        // Handle public-page interaction
    }
}

registry.category("public.interactions").add("my_module.my_public_interaction", MyPublicInteraction);
```

Use helpers from `@web/public/utils` where the public website layer exposes them instead of reusing old legacy DOM utilities.

---

## State Management

### useState

```javascript
setup() {
    this.state = useState({
        count: 0,
        name: "",
    });
}

increment() {
    this.state.count++;
}
```

### useState in Template

```xml
<div t-out="state.count"/>
<button t-on-click="increment">+</button>
```

### Computed State

```javascript
setup() {
    this.state = useState({count: 0});
    this.double = computed(() => this.state.count * 2);
}
```

---

## QWeb Templates

### Basic Template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="my_module.MyComponent">
        <div class="my_component">
            <h1 t-out="props.title"/>
            <p t-out="state.message"/>
        </div>
    </t>
</templates>
```

### Event Handlers

```xml
<button t-on-click="increment">Increment</button>
<input t-on-change="onChange"/>
```

### Loops and Conditions

```xml
<!-- Loop -->
<div t-foreach="state.records" t-as="record" t-key="record.id">
    <span t-out="record.name"/>
</div>

<!-- Condition -->
<div t-if="state.show">Visible when true</div>
<div t-else="">Visible when false</div>
```

---

## Translations

### Translate in JavaScript

```javascript
import { _t } from "@web/core/l10n/translation";

this.message = _t("Hello World");
```

### Translate with Parameters

```javascript
this.message = _t("Hello %(name)s", { name: "John" });
```

### Translate in Template

```xml
<span t-out="translate('Hello World')"/>
```

---

## Examples

### Counter Component

```javascript
import { Component, useState } from "@odoo/owl";
import { _t } from "@web/core/l10n/translation";

export class Counter extends Component {
  static template = "my_module.Counter";

  setup() {
    this.state = useState({ count: 0 });
  }

  increment() {
    this.state.count++;
  }

  decrement() {
    this.state.count--;
  }
}
```

```xml
<templates xml:space="preserve">
    <t t-name="my_module.Counter">
        <div class="counter">
            <button class="btn btn-secondary" t-on-click="decrement">-</button>
            <span t-out="state.count"/>
            <button class="btn btn-secondary" t-on-click="increment">+</button>
        </div>
    </t>
</templates>
```

### Data Loading Component

```javascript
import { Component, useState, onWillStart } from "@odoo/owl";
import { useService } from "@web/core/utils/hooks";

export class DataComponent extends Component {
  static template = "my_module.DataComponent";

  setup() {
    this.orm = useService("orm");
    this.state = useState({
      records: [],
      loading: true,
    });

    onWillStart(this.loadData);
  }

  async loadData() {
    this.state.records = await this.orm.search("my.model", [], {
      limit: 10,
    });
    this.state.loading = false;
  }
}
```

---

## Best Practices

### Use Hooks for Side Effects

```javascript
setup() {
    onMounted(() => {
        // Side effects here
    });
}
```

### Cleanup Resources

```javascript
setup() {
    onWillUnmount(() => {
        // Cleanup here
    });
}
```

### Avoid Direct DOM Manipulation

Use templates and reactive state instead.

### Split Components

Keep components small and focused.

```javascript
// Good: Small focused component
export class UserName extends Component {
  static template = "my_module.UserName";
}

// Bad: Large monolithic component
export class Everything extends Component {
  static template = "my_module.Everything";
}
```

---

## References

- OWL documentation
- Odoo 20 Web framework docs
