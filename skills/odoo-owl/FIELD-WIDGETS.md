# OWL Field Widgets

Use this reference when rendering or editing an Odoo field with a custom OWL
widget. Verify the target version's field registry, props, record API, and
template conventions against local source before coding.

## Workflow

1. Identify the field type, readonly behavior, and view contexts.
2. Inspect a nearby standard widget for the target Odoo version.
3. Start from `standardFieldProps` when the version exposes it.
4. Keep formatting and UI state in the component; keep business rules in the model.
5. Register the widget with the correct field registry category.
6. Test readonly, edit, empty, invalid, and relational values.

## Basic Widget Shape

```javascript
import { Component } from "@odoo/owl";
import { registry } from "@web/core/registry";
import { standardFieldProps } from "@web/views/fields/standard_field_props";

export class LabelField extends Component {
    static template = "my_module.LabelField";
    static props = {
        ...standardFieldProps,
        prefix: { type: String, optional: true },
    };

    get value() {
        return this.props.record.data[this.props.name] || "";
    }

    get readonly() {
        return this.props.readonly || !this.props.record.isEditable;
    }

    onInput(event) {
        if (!this.readonly) {
            this.props.record.update({
                [this.props.name]: event.target.value,
            });
        }
    }
}

registry.category("fields").add("label_field", {
    component: LabelField,
    supportedTypes: ["char", "text"],
    extractProps: ({ attrs }) => ({
        prefix: attrs.prefix,
    }),
});
```

The exact props and registry API are version-sensitive. Do not copy this shape
into an older Odoo version without checking its standard field widgets.

## Template

The following template uses the explicit `this` expression style used by
current OWL examples. OWL 2 projects may expose component values differently;
copy the expression context from a standard widget in the target source.

```xml
<templates xml:space="preserve">
    <t t-name="my_module.LabelField">
        <span t-if="this.readonly" class="o_my_label_field_readonly">
            <t t-out="this.props.prefix || ''"/><t t-out="this.value"/>
        </span>
        <input t-else=""
               class="o_my_label_field_input"
               type="text"
               t-att-value="this.value"
               t-on-input="this.onInput"
               t-att-aria-label="this.props.name"/>
    </t>
</templates>
```

Use escaped output by default. Keep labels, focus behavior, keyboard handling,
and validation states accessible. Do not use a widget to bypass field-level
security or server-side validation.

## Field Options

Use `extractProps` for XML options and normalize values before passing them to
the component. Avoid reading arbitrary attributes deep inside rendering code.
Keep option names documented and stable.

## Relational And Monetary Fields

- Read relational values using the record shape supplied by the target field model.
- Do not assume a Many2one value is always an ID, a tuple, or a record object.
- Use the standard monetary formatter and currency metadata rather than formatting numbers manually.
- Preserve `record.update()` semantics for x2many and relational edits.
- Reuse standard dropdown, autocomplete, and dialog components where available.

## Testing Checklist

- Renders a value in readonly mode.
- Renders an empty value without throwing.
- Does not mutate a readonly record.
- Updates the record with the expected field name and value.
- Accepts only the declared field types.
- Handles invalid or missing options safely.
- Loads its XML template and assets in the target bundle.
- Works in every view type where the widget is registered.
