# Odoo QWeb Patterns

## Template Inheritance

Inspect the parent template in the target Odoo version before writing an XPath. Prefer stable semantic anchors and the smallest possible insertion or replacement.

```xml
<template id="portal_my_record_inherit" inherit_id="portal.portal_my_home">
    <xpath expr="//div[@id='portal_common_category']" position="inside">
        <a href="/my/records" class="list-group-item list-group-item-action">
            <span class="fa fa-folder-open" aria-hidden="true"/>
            <span>My Records</span>
        </a>
    </xpath>
</template>
```

Verify the parent XML ID and resulting architecture. Do not copy a template from another Odoo version without checking its current structure.

## Safe Output

```xml
<h1 t-out="record.name"/>
<p t-if="record.description" t-out="record.description"/>
```

Use `t-out` for text and attributes. Do not use `t-raw` for ordinary user-controlled content. When intentional rich HTML is required, document its source, sanitization policy, and trust boundary.

## Forms

- Include a clear submit action and validation feedback.
- Preserve CSRF protection for state-changing HTTP forms according to the target version.
- Keep error messages translatable.
- Avoid putting authorization decisions only in template conditions.
- Use semantic labels, keyboard-accessible controls, and meaningful focus order.

## OWL Templates

For OWL templates, keep component state and event behavior in JavaScript while using QWeb for structure. Ensure the XML template is included in the same asset bundle as its component and use the version's supported directives.

## Review Checklist

- Parent template and XPath anchor exist in the target version.
- User data uses escaped output.
- Template conditions do not replace server-side authorization.
- Strings are translatable.
- Layout works in desktop, mobile, and RTL contexts where relevant.
