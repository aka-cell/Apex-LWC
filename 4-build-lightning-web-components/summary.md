
Yep. Here’s the **mental model of the whole unit**, rather than a pile of Salesforce vocabulary that evaporates the moment Trailhead closes.

## 1. Big picture

The unit is about **using Apex from a Lightning Web Component (LWC)**.

The flow is basically:

```text
User enters Annual Revenue
        ↓
LWC JavaScript property
annualRevenue
        ↓
@wire
        ↓
Apex method
queryAccountsByRevenue()
        ↓
SOQL query
        ↓
List<Account>
        ↓
LWC receives accounts.data
        ↓
HTML displays Account Names
```

The unit contrasts this with **Visualforce**, where Apex controllers work differently. 

---

# 2. Apex

**Apex** is Salesforce's server-side programming language.

You use it when standard Salesforce features or **Lightning Data Service** aren't enough, particularly for custom transactions or multi-record operations. 

Think:

```text
JavaScript/LWC → browser side
Apex          → Salesforce server side
```

---

# 3. Lightning Web Component (LWC)

An **LWC** is a reusable UI component built using:

* HTML
* JavaScript
* CSS
* Salesforce-specific features

Your `accountFinder` is an LWC.

It handles the UI:

```text
Input → Button → Display accounts
```

while Apex handles querying Salesforce data.

---

# 4. Apex Controller

An **Apex controller** is an Apex class containing methods that an LWC can call.

Example:

```apex
public with sharing class AccountListControllerLwc {
    ...
}
```

Your unit uses:

```text
AccountListControllerLwc
```

as the controller for `accountFinder`.

---

# 5. `@AuraEnabled`

This is an Apex annotation that makes an Apex method available to Lightning components, including LWCs.

Example:

```apex
@AuraEnabled(cacheable=true)
public static List<Account> queryAccountsByRevenue(...)
```

Without `@AuraEnabled`, the LWC can't call the method. 

### `cacheable=true`

Means Salesforce can cache the returned data on the client.

Useful when the method **only reads data** and doesn't modify records.

The unit notes that caching can prevent repetitive server calls, although changed records might not immediately appear until the cache is refreshed. 

---

# 6. `static`

A method called from an LWC needs to be `static`.

```apex
public static List<Account> queryAccountsByRevenue(...)
```

**Static** means the method belongs to the class itself rather than requiring an instance of the class.

The important thing for this unit:

> LWC-callable Apex methods are static and don't rely on instance state.

That makes them **stateless**. 

---

# 7. Stateless

This is an important concept.

In Visualforce, the controller can maintain state through **View State**.

In LWC + Apex, server calls are **stateless**.

Meaning:

```text
Request 1
  ↓
Apex gets everything it needs
  ↓
Returns result

Request 2
  ↓
Apex gets everything it needs again
```

Apex shouldn't assume it remembers what happened in the previous request.

Therefore, you pass required information as parameters:

```apex
queryAccountsByRevenue(annualRevenue)
```

The unit explicitly identifies stateless methods as one of the major differences between Visualforce and LWC. 

---

# 8. Parameter

A **parameter** is information passed into a method.

Example:

```apex
public static List<Account> queryAccountsByRevenue(Decimal annualRevenue)
```

Here:

```text
annualRevenue
```

is the parameter.

You're basically saying:

> "Give me an annual revenue value and I'll find accounts based on it."

The challenge specifically requires `annualRevenue` to be a **Decimal**. 

---

# 9. Decimal

`Decimal` is an Apex data type used for numbers that can contain decimal values.

For example:

```text
100000
250000.50
12345.75
```

It's appropriate for things like:

* Revenue
* Prices
* Currency
* Financial values

Hence:

```apex
Decimal annualRevenue
```

---

# 10. Account

**Account** is a standard Salesforce object.

It generally represents an organization/company/customer.

In Apex:

```apex
Account
```

is the Salesforce record type/object you're querying.

A collection of Accounts is:

```apex
List<Account>
```

---

# 11. List<Account>

A **List** is a collection of values.

```apex
List<Account>
```

means:

> "A list containing Account records."

For example:

```text
Account A
Account B
Account C
```

The Apex method returns this list:

```apex
return [
    SELECT Name
    FROM Account
    WHERE AnnualRevenue >= :annualRevenue
];
```

---

# 12. SOQL

**SOQL = Salesforce Object Query Language**

It's Salesforce's query language for retrieving records.

Example:

```apex
SELECT Name
FROM Account
WHERE AnnualRevenue >= :annualRevenue
```

Breaking it down:

```text
SELECT Name
       ↓
which field?

FROM Account
       ↓
which object?

WHERE AnnualRevenue >= ...
       ↓
which records?
```

---

# 13. Bind variable `:annualRevenue`

This little colon is important:

```apex
WHERE AnnualRevenue >= :annualRevenue
```

The `:` means:

> Use the Apex variable `annualRevenue` inside the SOQL query.

So if JavaScript gives Apex:

```text
annualRevenue = 500000
```

the query effectively becomes:

```text
AnnualRevenue >= 500000
```

You're passing a JavaScript/LWC value → Apex parameter → SOQL filter.

---

# 14. `@wire`

`@wire` is an LWC feature for getting data from Salesforce.

Example:

```javascript
@wire(queryAccountsByRevenue, {
    annualRevenue: '$annualRevenue'
})
accounts;
```

Think of it as:

> "Keep this Apex call connected to my component's data."

The unit explains that there are two ways to call Apex from LWC:

1. **Wire**
2. **Imperative call**



---

# 15. Wired Apex

When you do:

```javascript
@wire(queryAccountsByRevenue, {
    annualRevenue: '$annualRevenue'
})
accounts;
```

Salesforce automatically calls:

```text
queryAccountsByRevenue()
```

and puts the result into:

```text
accounts
```

Conceptually:

```text
annualRevenue changes
       ↓
@wire notices
       ↓
Apex runs
       ↓
accounts updates
```

That's **reactive data**.

---

# 16. Reactive variable

This:

```javascript
'$annualRevenue'
```

is important.

The `$` means:

> Treat `annualRevenue` as a reactive property.

So:

```javascript
annualRevenue = 100000;
```

changes to:

```javascript
annualRevenue = 500000;
```

and the wired Apex call reacts to that change.

Without the `$`, you're not telling the wire service to react to changes in that property.

---

# 17. `accounts`

This:

```javascript
accounts;
```

is where the result of the wired Apex call is stored.

It isn't the actual Account list directly.

The wire result has information such as:

```text
accounts.data
accounts.error
```

So:

```javascript
accounts.data
```

contains the successful result.

---

# 18. `accounts.data`

In HTML:

```html
<template lwc:if={accounts.data}>
```

means:

> Only render this section if Apex returned data.

Then:

```html
<template for:each={accounts.data} for:item="account">
```

means:

> Loop through every Account returned by Apex.

And:

```html
<p key={account.Id}>{account.Name}</p>
```

displays the Account's name.

The unit gives this exact pattern for displaying the returned accounts. 

---

# 19. `for:each`

This is LWC's template iteration syntax.

```html
<template for:each={accounts.data} for:item="account">
```

Think JavaScript:

```javascript
for (const account of accounts) {
    ...
}
```

But in LWC HTML.

---

# 20. `for:item`

This:

```html
for:item="account"
```

defines the variable representing the current item.

So if Salesforce returns:

```text
Google
Microsoft
Apple
```

each iteration gives you:

```text
account = Google
account = Microsoft
account = Apple
```

Then:

```html
{account.Name}
```

gets that Account's Name.

---

# 21. `key`

LWC requires a unique key when rendering lists:

```html
key={account.Id}
```

Salesforce uses the key to identify individual elements efficiently when the list changes.

`Id` works well because Salesforce record IDs are unique.

---

# 22. `lightning-input`

This is a Salesforce **base component**.

Instead of building an `<input>` yourself, you use:

```html
<lightning-input>
```

Your challenge configures it as:

```html
type="number"
label="Annual Revenue"
formatter="currency"
```

So the user sees a currency-formatted numerical input.

---

# 23. Event handler

An **event handler** is JavaScript that runs when something happens in the UI.

Your input:

```html
onchange={handleChange}
```

means:

```text
User changes input
        ↓
change event
        ↓
handleChange()
```

Your button:

```html
onclick={reset}
```

means:

```text
User clicks button
        ↓
click event
        ↓
reset()
```

---

# 24. `event`

When the event happens, Salesforce gives your handler an **event object**.

Example:

```javascript
handleChange(event) {
    ...
}
```

That object contains information about what happened.

In your component, you're interested in the input's value.

---

# 25. `event.detail.value` vs `event.target.value`

This was the annoying part of your previous challenge.

For Salesforce base components such as `lightning-input`, you may encounter:

```javascript
event.target.value
```

or component event details depending on the component/event involved.

For the challenge you were working through, the accepted implementation was:

```javascript
this.annualRevenue = event.detail.value;
```

The key concept is:

```text
Event
 ↓
value
 ↓
annualRevenue property
```

---

# 26. Property

A property is data stored on your LWC JavaScript class.

Example:

```javascript
annualRevenue = null;
```

This component has a property called:

```text
annualRevenue
```

Its initial value is:

```text
null
```

Then when the user enters something:

```text
annualRevenue = 500000
```

That property drives the Apex wire.

---

# 27. `reset()`

Your reset method:

```javascript
reset() {
    this.annualRevenue = null;
}
```

sets the property back to `null`.

Because `annualRevenue` is used reactively by `@wire`, changing it also affects the wired Apex call.

---

# 28. `this`

In:

```javascript
this.annualRevenue
```

`this` refers to the current LWC component instance.

So:

```javascript
this.annualRevenue
```

means:

> "The `annualRevenue` property belonging to this component."

---

# 29. Imperative Apex call

The other way to call Apex is **imperatively**.

Instead of:

```javascript
@wire(...)
```

you explicitly call the imported Apex method:

```javascript
queryAccountsByRevenue({ annualRevenue: this.annualRevenue })
```

This gives you more control over exactly **when** the request happens.

The unit contrasts imperative calls with wired calls. 

---

# 30. `recordId`

Another concept in the unit is accessing the Salesforce record currently being displayed.

In an LWC, you can expose:

```javascript
@api recordId;
```

The Lightning record page supplies the current record's ID.

Then you pass it into Apex:

```text
Record Page
    ↓
recordId
    ↓
LWC
    ↓
Apex
```

The component must be exposed to record pages in its metadata XML. 

---

# 31. `@api`

`@api` makes an LWC property **public**.

Example:

```javascript
@api recordId;
```

A parent component or Lightning page can provide that value.

Think:

```text
Private property
annualRevenue

Public property
@api recordId
```

---

# 32. Metadata XML

Every LWC has a metadata file:

```text
accountFinder.js-meta.xml
```

It controls things like:

* Whether the component is exposed
* Where it can be used
* Which pages can contain it

Example:

```xml
<isExposed>true</isExposed>
```

means the component can be exposed to Salesforce builders.

And:

```xml
<target>lightning__RecordPage</target>
```

means it can be placed on record pages.

The unit demonstrates this with `accountDetails.js-meta.xml`. 

---

# 33. Visualforce vs LWC + Apex

This is the central comparison of the unit.

| Visualforce                               | LWC                             |
| ----------------------------------------- | ------------------------------- |
| Apex controller                           | Apex class/method               |
| View State                                | Stateless server calls          |
| Controller properties maintain state      | Pass data as parameters         |
| Apex methods exposed through controller   | Apex methods use `@AuraEnabled` |
| Page references controller                | JS imports Apex method          |
| `ApexPages` messages                      | JS handles errors               |
| Standard controller can provide record ID | LWC uses `@api recordId`        |

The unit explicitly frames these as the major differences. 

---

# 34. The whole `accountFinder` example

Now connect everything:

### HTML

```text
lightning-input
       ↓
onchange
       ↓
handleChange()
       ↓
annualRevenue
```

### JavaScript

```text
annualRevenue
       ↓
@wire
       ↓
queryAccountsByRevenue
       ↓
accounts
```

### Apex

```text
queryAccountsByRevenue(annualRevenue)
       ↓
SOQL
       ↓
SELECT Name FROM Account
WHERE AnnualRevenue >= :annualRevenue
       ↓
List<Account>
```

### Back to HTML

```text
accounts.data
       ↓
for:each
       ↓
account.Name
       ↓
displayed on screen
```

That's the **core architecture** you should remember.

## The terms worth memorizing

If you're studying this for Salesforce work/interviews, prioritize these:

**LWC → Apex → `@AuraEnabled` → `static` → stateless → SOQL → parameter → `@wire` → reactive `$property` → `accounts.data` → `for:each` → `@api` → `recordId` → imperative Apex → metadata XML.**

Those are the concepts doing the actual work here. The rest is mostly Salesforce naming ceremony, because apparently software platforms need their own dialect before they'll let you query an Account.
