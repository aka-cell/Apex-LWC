
# Add Styles and Data to an LWC

This unit adds two important capabilities to the Bike Selector:

1. **Styling** the component.
2. **Getting live data from Salesforce** instead of relying only on static data. 

## 1. CSS and Component Styling

An LWC can have a `.css` file that automatically applies to its corresponding `.html` file.

For example:

```css
.price {
    color: green;
    font-weight: bold;
}
```

This changes how the price looks without changing the JavaScript or HTML logic.

### Shadow DOM

LWC uses **Shadow DOM** to encapsulate a component.

Think of it as giving each component its own little DOM world:

```text
Component A
  └── its HTML + CSS

Component B
  └── its HTML + CSS
```

This separation helps prevent a component's styling and behavior from accidentally interfering with other components.

**Mental model:**
**HTML = structure, JS = behavior/data, CSS = appearance.**

---

## 2. Salesforce Lightning Design System (SLDS)

**SLDS** is Salesforce's CSS framework for making components look consistent with Lightning Experience.

For example:

```html
<div class="slds-text-heading_small">
    {product.fields.Name.value}
</div>
```

Instead of creating your own heading style, you use a predefined SLDS class.

The important benefit is **consistency with Salesforce's existing UI**.

The material also notes that components running in Lightning Experience or the Salesforce mobile app can use SLDS without needing imports or static resources.

So there are two styling approaches here:

* **Your CSS** → custom appearance for your component.
* **SLDS classes** → Salesforce's established visual language.

---

# 3. From Static Data to Salesforce Data

Previously, the bike selector used a local data file.

Now the component gets data from the actual Salesforce org.

This is where the **wire service** comes in.

> **Wire service = a mechanism that delivers Salesforce data reactively to an LWC.**

It is built on **Lightning Data Service**.

---

## 4. `@wire`

To use the wire service, you generally:

1. Import a **wire adapter**.
2. Use `@wire` on a property or function.

Basic pattern:

```js
import { adapterId } from 'adapter-module';

@wire(adapterId, adapterConfig)
propertyOrFunction;
```

### The pieces

**Wire adapter**
The thing that knows how to retrieve the desired data.

**Adapter configuration**
Tells the adapter what data you want.

**Property/function**
Receives the result.

If you wire to a **property**, the result contains:

```text
data
error
```

So conceptually:

```text
@wire
  ↓
wire adapter
  ↓
Salesforce data
  ↓
property.data / property.error
```

---

# 5. The Example: Getting the Current User's Name

The selector uses:

```js
import { getRecord, getFieldValue } from 'lightning/uiRecordApi';
import Id from '@salesforce/user/Id';
import NAME_FIELD from '@salesforce/schema/User.Name';
```

The important pieces are:

* `Id` → identifies the current user.
* `NAME_FIELD` → identifies the `User.Name` field.
* `getRecord` → retrieves the record.
* `getFieldValue` → extracts the field's value.

Then:

```js
@wire(getRecord, { recordId: '$userId', fields })
user;
```

This says:

> "Use `getRecord` to retrieve this user's record and give the result to `user`."

Then the getter:

```js
get name() {
    return getFieldValue(this.user.data, NAME_FIELD);
}
```

extracts the user's name.

Finally HTML can simply use:

```html
<header>Available Bikes for {name}</header>
```

So the flow is:

```text
Current User ID
      ↓
   getRecord
      ↓
Salesforce User record
      ↓
getFieldValue()
      ↓
    name
      ↓
   HTML UI
```

The important conceptual shift is:

**Static data:** component gets information from code/data files.

**Dynamic data:** component gets information from Salesforce through the wire service.

---

# 6. Mobile-Friendly Markup

The final section makes a broader development point.

The Bike Selector works, but its markup wasn't designed with mobile in mind. Retrofitting mobile support later can require significantly more work.

SLDS and mobile-preview tools can help, but the deeper lesson is:

> **Consider the target experience early rather than assuming you'll easily fix it later.**

The Trailhead material deliberately uses its own example to demonstrate this lesson.

---

# Final Mental Model

This unit expands your LWC from merely **working** to being more **Salesforce-aware**:

```text
LWC
├── HTML → structure
├── JS → logic + data
├── CSS → custom styling
├── SLDS → Salesforce-consistent styling
└── @wire → Salesforce data
```

And the biggest new concept is:

> **`@wire` connects your component to Salesforce data reactively, while CSS and SLDS control how that data is presented.**

That completes the basic progression of the module: **build → compose → communicate → style → connect to Salesforce data.**
