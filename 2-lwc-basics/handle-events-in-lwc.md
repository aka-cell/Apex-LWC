
# Handle Events in Lightning Web Components

This unit is really about one central idea:

> **Components communicate by sending events upward and passing properties downward.**

The bike selector is useful because it shows that idea across several nested components. 

## 1. Component Composition

The app is divided into four components:

* **`tile`** → displays one bike.
* **`list`** → displays multiple tiles.
* **`detail`** → displays information about the selected bike.
* **`selector`** → acts as the overall container.

The hierarchy is roughly:

```text
selector
├── list
│   ├── tile
│   ├── tile
│   └── tile
└── detail
```

This is **component composition**: building a larger UI by nesting smaller components inside one another.

The important part is that the components aren't isolated. Their **parent-child relationships determine how they communicate**.

---

## 2. Events Up, Properties Down

This is the key pattern to remember:

```text
             EVENT ↑
Child  ─────────────────→ Parent

             PROPERTY ↓
Parent ─────────────────→ Child
```

### Passing information up

A child uses an **event** to tell its parent that something happened.

For example:

```js
this.dispatchEvent(new CustomEvent('next'));
```

The child says essentially:

> "Something called `next` just happened."

The parent listens:

```html
<c-todo-item onnext={nextHandler}></c-todo-item>
```

and responds in JavaScript.

### Passing information down

A parent passes information to a child through a **public property**:

```js
@api itemName;
```

Then the parent provides the value:

```html
<c-todo-item item-name="Milk"></c-todo-item>
```

Notice the naming difference:

```text
JavaScript: itemName
HTML:       item-name
```

JavaScript uses **camelCase**; HTML attributes use **kebab-case**.

---

## 3. `@api` Means Public

`@api` is important because it makes a property or method **publicly accessible to the parent**.

### Public property

```js
@api itemName;
```

Parent:

```html
<c-todo-item item-name="Milk"></c-todo-item>
```

### Public method

A child can also expose a method:

```js
@api play() {
   // ...
}
```

The parent can then call it:

```js
this.template.querySelector('c-video-player').play();
```

So:

* `@api property` → parent **gives data to child**
* `@api method` → parent **asks child to perform something**

---

# 4. The Bike Selector Event Journey

This is the most important part of the unit.

When the user clicks a bike:

```text
tile
 ↓ event
list
 ↓ event
selector
 ↓ property
detail
```

Let's follow it.

### Step 1: User clicks the tile

`tile.html` listens for the browser's click:

```html
<a onclick={tileClick}>
```

That calls `tileClick()` in `tile.js`.

### Step 2: Tile creates an event

```js
const event = new CustomEvent('tileclick', {
    detail: this.product.fields.Id.value
});

this.dispatchEvent(event);
```

The important pieces are:

* `CustomEvent('tileclick')` → creates a custom event.
* `detail` → carries the selected bike's ID.
* `dispatchEvent()` → sends the event upward.

Think:

> **The tile says: "The user selected bike ID 123."**

---

### Step 3: List catches the event

The parent listens using `on` + event name:

```html
<c-tile ontileclick={handleTileClick}>
```

So:

```text
tileclick
    ↓
ontileclick
    ↓
handleTileClick()
```

The `list` component then receives the event and dispatches another event, `productselected`, carrying the same ID upward.

This is **event propagation through the component hierarchy**.

---

### Step 4: Selector catches it

`selector.html` listens:

```html
<c-list onproductselected={handleProductSelected}>
```

Its handler gets the ID:

```js
this.selectedProductId = evt.detail;
```

Now the selector knows which bike was selected.

---

### Step 5: Selector passes the ID down

The selector gives that ID to the detail component:

```html
<c-detail product-id={selectedProductId}></c-detail>
```

Notice what happened:

```text
Event travels UP
tile → list → selector

Property travels DOWN
selector → detail
```

This is the entire architecture of the example.

---

## 5. Why the Getter/Setter Exists

The `detail` component receives `productId`, but it needs to turn that ID into the actual bike object.

The setter does that work:

```js
set productId(value) {
    this._productId = value;
    this.product = bikes.find(
        bike => bike.fields.Id.value === value
    );
}
```

So when the parent sends a new ID:

```text
productId = selected ID
        ↓
setter runs
        ↓
find matching bike
        ↓
product gets that bike
        ↓
detail.html displays it
```

The getter:

```js
@api get productId() {
    return this._productId;
}
```

provides access to the stored value.

**Mental model:** a setter isn't merely storing a value here. It's saying:

> "Whenever `productId` changes, do some work to update the component's state."

---

## 6. Conditional Rendering

`detail.html` uses:

```html
<template lwc:if={product}>
```

If `product` exists, show the bike.

Otherwise:

```html
<template lwc:else>
    <div>Select a bike</div>
</template>
```

So initially:

```text
No product selected → "Select a bike"
```

After clicking:

```text
Product found → show bike details
```

---

# The Big Picture

The whole interaction can be remembered as:

```text
USER CLICKS BIKE
       ↓
     tile
       │
       │  event: tileclick
       ↓
     list
       │
       │  event: productselected
       ↓
   selector
       │
       │  product-id
       ↓
    detail
       ↓
  display bike
```

### Key takeaway

**LWC communication follows "Events Up, Properties Down."**

* **Child → Parent:** `CustomEvent` + `dispatchEvent()`
* **Parent listens:** `on<eventname>`
* **Parent → Child:** public `@api` properties or methods
* **`detail`** carries information inside a custom event.
* **Getters/setters** can add logic when a public property is received.

If you remember only one thing from this unit, remember the arrows:

> **Events go UP. Data/properties go DOWN.**

Humanity has spent considerable effort inventing complicated communication systems. LWC at least has the decency to draw the arrows for us.
