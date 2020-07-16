Slots

# [Overview](https://developer.salesforce.com/docs/component-library/documentation/en/lwc/create_components_slots)

A placeholder for markup that a parent component passes into a component’s body.

## Examples

### Unnamed

> slotDemo.html

```html
<!-- slotDemo.html -->
<template>
    <h1>Add content to slot</h1>
    <div>
        <slot></slot>
    </div>
</template>
```

> slotWrapper.html

```html
<!-- slotWrapper.html -->
<template>
    <c-slot-demo>
        <p>content from parent</p>
    </c-slot-demo>
</template>
```

### Named

> namedSlots.html

```html
<!-- namedSlots.html -->
<template>
    <p>First Name: <slot name="firstName">Default first name</slot></p>
    <p>Last Name: <slot name="lastName">Default last name</slot></p>
    <p>Description: <slot>Default description</slot></p>
</template>
```

> slotWrapper.html

```html
<!-- slotsWrapper.html -->
<template>
    <c-named-slots>
        <span slot="firstName">Willy</span>
        <span slot="lastName">Wonka</span>
        <span>Chocolatier</span>
    </c-named-slots>
</template>
```

## Selecting Rendered Elements

Use **this.querySelector()** and **this.querySelectorAll()**

```javascript
// namedSlots.js
import { LightningElement } from 'lwc';

export default class NamedSlots extends LightningElement {
    renderedCallback() {
        this.querySelector('span'); // <span>push the green button.</span>
        this.querySelectorAll('span'); // [<span>push the green button</span>, <span>push the red button</span>]
    }
}
```

## Events

- **slotchange** - fires when the direct child of a node in a slot changes

