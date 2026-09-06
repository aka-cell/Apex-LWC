
> This project is essentially teaching you **how Lightning Web Components (LWC) talk to each other**.

Think of each LWC as an independent person. Salesforce gives them different ways to communicate depending on their relationship:

```text
                 Parent
                /      \
           Child A    Child B
           
      Unrelated Component
```

The key question is:

> **Who needs to send information to whom?**

That determines the communication pattern.

---

# 1. The Three Communication Patterns

There are three important cases:

| Situation                                 | Communication | Main mechanism                               |
| ----------------------------------------- | ------------- | -------------------------------------------- |
| Child → Parent                            | Upward        | **Custom Events**                            |
| Parent → Child                            | Downward      | **Public properties / methods** using `@api` |
| Unrelated Component → Unrelated Component | Across        | **Lightning Message Service (LMS)**          |

So remember this:

```text
Child ───────► Parent
       Event

Parent ───────► Child
        @api

Component A ───────► Component B
              LMS
```

This is the central idea of the whole project.

---

# 2. Parent → Child Communication

This is the first topic in your objectives.

Suppose we have:

```text
Parent Component
      │
      ▼
Child Component
```

The parent knows about the child because the child is placed inside the parent's HTML.

For example:

```html
<c-child></c-child>
```

The parent can therefore pass information to the child.

## The mechanism: `@api`

In LWC, a property or method marked with `@api` becomes **public**.

That means:

> "Other components are allowed to interact with this."

### Child JavaScript

```javascript
import { LightningElement, api } from 'lwc';

export default class Child extends LightningElement {
    @api message;
}
```

The child now has a public property called `message`.

The parent can provide a value:

### Parent HTML

```html
<c-child message="Hello Child"></c-child>
```

The child receives:

```text
message = "Hello Child"
```

So the flow is:

```text
Parent HTML
     │
     │ message="Hello Child"
     ▼
Child @api message
```

---

# 3. A More Realistic Example

Imagine a parent component displaying an employee's information.

The child component displays the employee's name.

### Parent

```html
<c-employee-card employee-name="John"></c-employee-card>
```

### Child

```javascript
import { LightningElement, api } from 'lwc';

export default class EmployeeCard extends LightningElement {
    @api employeeName;
}
```

### Child HTML

```html
<lightning-card title={employeeName}>
</lightning-card>
```

Result:

```text
Parent
  │
  │ employeeName = "John"
  ▼
Employee Card
  │
  ▼
Displays "John"
```

The parent is essentially saying:

> "Here is some data. You display it."

---

# 4. Parent Can Also Call a Child's Method

`@api` isn't limited to properties.

You can expose a **method** from the child.

### Child

```javascript
import { LightningElement, api } from 'lwc';

export default class Child extends LightningElement {

    @api
    showMessage() {
        console.log('Hello from parent!');
    }
}
```

Because `showMessage()` has `@api`, the parent can call it.

### Parent JavaScript

```javascript
const child = this.template.querySelector('c-child');
child.showMessage();
```

So:

```text
Parent
   │
   │ calls showMessage()
   ▼
Child
```

This is useful when the parent wants the child to **perform an action**, rather than simply receive data.

---

# 5. When Should You Use Parent → Child?

Use it when:

* the parent owns the information
* the child needs that information
* the parent needs to tell the child to perform something

For example:

```text
Parent
 │
 ├── Customer Name
 ├── Customer Email
 └── Customer Status
        │
        ▼
     Child
```

The parent can pass these values to the child.

### Mental model

**Parent → Child = "Here's something you need."**

---

# 6. Child → Parent Communication

Now reverse the direction:

```text
Parent
   ▲
   │
   │
 Child
```

This is slightly different.

A child **doesn't directly modify the parent's properties**.

Instead, the child says:

> "Something happened."

The parent listens and decides what to do.

The mechanism is a **Custom Event**.

---

## Example: Button in Child

Suppose the child has a button:

```html
<lightning-button
    label="Select"
    onclick={handleClick}>
</lightning-button>
```

When clicked, the child dispatches an event.

### Child JavaScript

```javascript
handleClick() {
    this.dispatchEvent(new CustomEvent('selected'));
}
```

The important part is:

```javascript
this.dispatchEvent(...)
```

The child is broadcasting an event.

---

## Parent Listens

The parent puts the child in its HTML:

```html
<c-child onselected={handleSelected}></c-child>
```

Notice:

```text
onselected
```

This corresponds to:

```javascript
new CustomEvent('selected')
```

When the child fires the event:

```text
Child
 │
 │ dispatchEvent('selected')
 ▼
Parent
 │
 ▼
handleSelected()
```

### Parent JavaScript

```javascript
handleSelected() {
    console.log('Child was selected');
}
```

---

# 7. Passing Data with the Event

Events become much more useful when the child sends information.

Suppose the child wants to tell the parent which product was selected.

### Child

```javascript
handleClick() {
    this.dispatchEvent(
        new CustomEvent('productselected', {
            detail: {
                productId: '001ABC'
            }
        })
    );
}
```

The data is stored in:

```javascript
detail
```

The parent receives the event:

```javascript
handleProductSelected(event) {
    console.log(event.detail.productId);
}
```

So:

```text
Child
 │
 │ CustomEvent
 │
 │ detail = { productId: "001ABC" }
 ▼
Parent
 │
 ▼
event.detail.productId
```

### Mental model

**Child → Parent = "Hey, something happened, and here's the information."**

---

# 8. Why Not Just Change the Parent Directly?

Because LWC encourages **controlled communication**.

Instead of:

```text
Child directly changes Parent
```

you use:

```text
Child
  │
  │ Event
  ▼
Parent
  │
  │ decides what to do
  ▼
Updated state
```

This keeps components more independent and easier to understand.

---

# 9. Communicating Between Unrelated Components

Now we reach the more interesting problem.

Imagine:

```text
Component A          Component B
     │                    │
     │                    │
     └────── ??? ─────────┘
```

They don't have a parent-child relationship.

For example:

```text
             App
          /       \
         /         \
Component A      Component B
```

Component A needs to tell Component B something.

You can't conveniently use:

```text
@api
```

because `@api` is designed for component relationships such as parent → child.

You also can't simply use a normal child event because Component B isn't the parent.

This is where **Lightning Message Service (LMS)** comes in.

---

# 10. Lightning Message Service

Think of LMS as a **communication channel**.

```text
Component A
     │
     │ publishes message
     ▼
 Message Channel
     │
     │
     ▼
Component B
     │
     │ receives message
```

The components don't need to know each other directly.

They communicate through the channel.

---

# 11. Message Channel

A **Message Channel** defines the communication channel.

Conceptually:

```text
Message Channel
       │
       ├── Component A publishes
       │
       └── Component B subscribes
```

The message might contain:

```javascript
{
    recordId: '001ABC',
    action: 'refresh'
}
```

Component A publishes it.

Component B receives it.

---

# 12. Publisher and Subscriber

Two terms are important.

### Publisher

The component that **sends** the message.

```text
Component A
     │
     ▼
  Publisher
```

### Subscriber

The component that **receives** the message.

```text
 Message Channel
       │
       ▼
 Component B
  Subscriber
```

A component can also be both.

---

# 13. Real-World Example

Imagine a Salesforce page containing:

```text
┌─────────────────────────────────┐
│ Product List                    │
│                                 │
│ [Product A] [Product B]         │
└─────────────────────────────────┘

┌─────────────────────────────────┐
│ Product Details                 │
│                                 │
│ Selected Product: Product B     │
└─────────────────────────────────┘
```

The product list and product details components are unrelated.

When the user selects Product B:

```text
Product List
     │
     │ publishes Product B
     ▼
Lightning Message Channel
     │
     ▼
Product Details
     │
     ▼
Displays Product B
```

That's exactly the kind of problem LMS solves.

---

# 14. The Big Picture

You should now see the three patterns as one system:

### Child → Parent

```text
       Parent
         ▲
         │ Custom Event
         │
       Child
```

Use:

```javascript
this.dispatchEvent(new CustomEvent(...))
```

---

### Parent → Child

```text
       Parent
         │
         │ @api
         ▼
       Child
```

Use:

```javascript
@api property;
```

or

```javascript
@api method() {}
```

---

### Unrelated → Unrelated

```text
Component A
     │
     │ publish
     ▼
Message Channel
     │
     │ subscribe
     ▼
Component B
```

Use:

**Lightning Message Service**

---

# 15. How to Decide Which One to Use

When building an LWC, ask one question first:

> **What is the relationship between the components?**

### If it's parent → child:

Use:

**`@api`**

```text
Parent ──► Child
```

### If it's child → parent:

Use:

**Custom Event**

```text
Child ──► Parent
```

### If they're unrelated:

Use:

**Lightning Message Service**

```text
A ──► Message Channel ──► B
```

That's the core of this entire project.

---

# 16. The Most Important Mental Model

Don't memorize random syntax first. Remember the **direction of communication**:

```text
             PARENT
            ↙      ↘
       Event       @api
        ↙            ↘
     CHILD          CHILD


      UNRELATED COMPONENTS

        A ───────► B
             LMS
```

Or even more simply:

> **Child speaks upward with events.**
> **Parent speaks downward with `@api`.**
> **Unrelated components speak through Lightning Message Service.**

Once that clicks, the code becomes much less mysterious. Salesforce has merely invented several ceremonious ways for two boxes on a screen to say, "Hey, I have some data."
