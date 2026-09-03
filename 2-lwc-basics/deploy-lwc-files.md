
## Deploy Lightning Web Component Files

The main idea of this unit is simple:

**You have an LWC in your local project. Now you need to make Salesforce recognize it, deploy it to an org, and place it on a page.** 

### 1. The three important component files

A basic LWC here consists of:

* `bikeCard.html` → **what the component displays**
* `bikeCard.js` → **the component's data/logic**
* `bikeCard.js-meta.xml` → **how Salesforce should expose and configure the component**

No CSS is needed because this component doesn't define custom styling.

The third file is the important new piece. You may have been happily ignoring it in LWC Studio, but Salesforce, being Salesforce, wants metadata before it lets your component into the building.

---

### 2. The `.js-meta.xml` configuration file

This file tells Salesforce **how and where the component can be used**.

The key parts are:

* **`apiVersion`** → associates the component with a Salesforce API version.
* **`isExposed`** → controls whether the component can be exposed to builders.

  * `true` → available for use in Lightning App Builder/Experience Builder when targets are defined.
  * `false` → not exposed there.
* **`targets`** → specifies which Lightning page types can contain the component.

  * `lightning__AppPage`
  * `lightning__RecordPage`
  * `lightning__HomePage`
* **`targetConfigs`** → optional, lets you add configuration specific to a target, such as restricting a component to particular objects.

So this:

```xml
<isExposed>true</isExposed>
<targets>
    <target>lightning__AppPage</target>
    <target>lightning__RecordPage</target>
</targets>
```

essentially tells Salesforce:

> "This component is allowed to appear in these kinds of Lightning pages."

**Important distinction:**

`isExposed` answers **"Can builders see/use it?"**

`targets` answers **"Where can they use it?"**

---

### 3. Getting the component into the org

Once the files are ready, you **deploy** them from VS Code to your Salesforce org.

The basic flow is:

**Local LWC files → Authenticate with org → Deploy source → Component exists in org**

Authentication establishes the connection between your local Salesforce project and the org. Deployment then sends the component files to Salesforce.

---

### 4. Why the Trusted URL is needed

The bike's image is hosted externally on an Amazon AWS URL.

Salesforce doesn't automatically trust every external resource, so the AWS domain must be added to **Trusted URLs** with `img-src` enabled.

The important concept isn't the particular AWS address. It's:

> **External resources used by the component may need to be explicitly trusted by Salesforce.**

---

### 5. Putting the component on a Lightning page

Because the metadata file declares `lightning__AppPage` as a target, the component becomes available in **Lightning App Builder**.

The flow is:

**`isExposed = true` + target defined**
↓
**Component appears in Lightning App Builder**
↓
**Drag component onto page**
↓
**Save + Activate**
↓
**Component appears in the org UI**

There is another approach mentioned in the material: using a **tab that points to an Aura component containing the LWC**. But the App Builder approach is the simpler one used here.

---

### Mental model

Think of deploying an LWC as a **three-layer process**:

**1. Build it**
`HTML + JS`

**2. Tell Salesforce where it belongs**
`.js-meta.xml`

**3. Send it to Salesforce and place it on a page**
`Deploy → App Builder → Activate`

The most important thing to remember is that **the `.js-meta.xml` file is the bridge between your LWC code and Salesforce's page-building system.**
