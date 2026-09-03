
# ⚡ Create Lightning Web Components

This unit is about **how an LWC is actually built and how its JavaScript controls what happens inside the component**. 

## 1. Basic Structure of an LWC

A component is essentially a folder containing files with the same name:

```text
app/
├── app.html
├── app.js
└── app.css   (optional)
```

* **HTML** → structure/UI
* **JavaScript** → data, logic, events
* **CSS** → styling

The HTML uses `{propertyName}` to display values from the JavaScript class.

For example:

```html
<div>Name: {name}</div>
```

gets its `name` from JavaScript:

```js
name = 'Electra X4';
```

So:

**JavaScript provides the data → HTML displays it.**

---

## 2. Conditional Rendering

You can decide what appears on screen using:

```html
<template lwc:if={ready}>
```

and:

```html
<template lwc:else>
```

For example:

```text
ready = false
     ↓
"Loading..."

ready = true
     ↓
Bike information
```

This is useful when data takes time to load.

---

## 3. Base Lightning Components

You don't have to build everything yourself.

Salesforce provides ready-made components such as:

```html
<lightning-badge label={category}></lightning-badge>
```

So instead of creating your own badge with HTML and CSS, you can reuse Salesforce's **base Lightning web components**.

The general idea is:

> **Build what is unique to your application, reuse what Salesforce already provides.**

---

## 4. Component Naming

A component folder might be:

```text
myComponent
```

but when you reference it in HTML, it becomes:

```html
<c-my-component>
```

That's because Salesforce converts **camelCase → kebab-case** in markup.

Also, `c` represents the default custom-component namespace.

---

# 5. JavaScript: Where the Component Actually Does Things

Every LWC JavaScript file extends `LightningElement`:

```js
import { LightningElement } from 'lwc';

export default class MyComponent extends LightningElement {
}
```

Think of `LightningElement` as the **base class that gives your JavaScript class the functionality of an LWC**.

The `lwc` module provides Salesforce's LWC functionality.

---

# 6. Lifecycle Hooks

A component goes through a lifecycle:

```text
Created
   ↓
Added to DOM
   ↓
Rendered
   ↓
Updated
   ↓
Removed
```

**Lifecycle hooks** let your JavaScript respond to these events.

The important one introduced here is:

```js
connectedCallback()
```

It runs when the component is inserted into the DOM.

For example:

```js
connectedCallback() {
    this.ready = true;
}
```

means:

> "When this component enters the page, execute this code."

There's also:

```js
disconnectedCallback()
```

which runs when the component is removed from the DOM. 

---

# 7. Decorators

Decorators modify or expose properties/functions.

The three important ones here are:

### `@api`

Makes a property **public**, allowing another component to pass data into it.

```js
@api bike;
```

Think:

```text
Parent component
      │
      │ bike={bike}
      ▼
Child component
      │
      └── @api bike
```

So `@api` is mainly about **communication from a parent component to a child component**.

---

### `@track`

Used when you want the framework to observe changes **inside an object or array**.

Modern LWC makes normal fields reactive automatically, so you generally don't need `@track` for simple values.

---

### `@wire`

Provides a way to **retrieve and bind Salesforce data** to your component.

So the quick memory trick is:

```text
@api   → Public property / component communication
@track → Observe changes inside objects/arrays
@wire  → Get Salesforce data
```



---

# 🧠 The Whole Unit in One Picture

```text
             LWC
              │
      ┌───────┼────────┐
      ↓       ↓        ↓
    HTML   JavaScript  CSS
     │         │       │
   UI       Logic    Styling
               │
        ┌──────┼───────┐
        ↓      ↓       ↓
    Lifecycle @api   @wire
     Hooks           Salesforce
        │
        ↓
  Component behavior
```

### The key idea

**HTML describes what the component looks like. JavaScript controls its data and behavior. Lifecycle hooks let JavaScript react to the component's life. Decorators give the component additional capabilities such as public properties and Salesforce data access.**

That's really the heart of this unit.
