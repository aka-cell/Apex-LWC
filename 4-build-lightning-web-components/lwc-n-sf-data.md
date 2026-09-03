
# Use Lightning Data Service (LDS) to Work with Data

This unit is about **how an LWC reads and changes Salesforce data without immediately reaching for Apex**.

The main idea is to choose the **simplest data solution that gives you enough control**. 

## 1. Lightning Data Service

**Lightning Data Service (LDS)** is the preferred, easiest way in this material to work with Salesforce data from an LWC.

It provides two major benefits:

* **Security** through Salesforce's secure UI API endpoints.
* **Performance** through a shared client-side cache.

The cache is particularly important:

```text
Component A ──┐
Component B ──┼── LDS cache ── Salesforce
Component C ──┘
```

If one component updates a record through LDS, other components using that record can receive the updated value.

So LDS isn't just "a way to fetch records." It also helps **keep data synchronized across components**.

---

# 2. Three Record Form Components

For working with a **single record**, the easiest options are the `lightning-record-*-form` components.

| Component                    | Purpose           |
| ---------------------------- | ----------------- |
| `lightning-record-form`      | View **and** edit |
| `lightning-record-view-form` | View only         |
| `lightning-record-edit-form` | Edit only         |

### `lightning-record-form`

It's the simplest and most declarative option.

You tell it:

* which object
* which fields
* optionally which record

and Salesforce handles much of the form behavior.

An especially important detail:

> If `lightning-record-form` has **no `recordId`**, it operates in edit mode and can create a new record when submitted.

### When to use which?

Use `lightning-record-form` when you want simplicity.

Use `lightning-record-view-form` or `lightning-record-edit-form` when you need **more control over the layout, prepopulated values, or rendering**.

---

# 3. Why Import Salesforce Schema References?

The example imports:

```js
import ACCOUNT_OBJECT from '@salesforce/schema/Account';
import NAME_FIELD from '@salesforce/schema/Account.Name';
```

Instead of simply writing strings like `"Account.Name"`.

These are **schema references**, and the material highlights their **referential integrity** benefits.

Salesforce can verify that the referenced object and fields exist and maintain those references when changes happen.

So mentally:

> **Schema imports = safer, Salesforce-aware references to objects and fields.**

---

# 4. LDS Wire Adapters: Reading Data

When record forms aren't customizable enough, you can work with LDS more directly.

For **reading** Salesforce data, use **LDS wire adapters**.

Example:

```js
@wire(getRecord, {
    recordId: '$recordId',
    fields: [ACCOUNT_NAME_FIELD]
})
account;
```

Here:

* `getRecord` = wire adapter
* `recordId` = which record to retrieve
* `fields` = what to retrieve
* `account` = receives the result

The result can contain:

```text
account.data
account.error
```

### The `$` is important

```js
recordId: '$recordId'
```

The `$` makes the parameter **reactive**.

So if `recordId` changes:

```text
recordId changes
      ↓
@wire reacts
      ↓
new data requested
      ↓
cache or server
      ↓
account updated
```

This is one of the important Trailhead details to remember.

---

# 5. Wire a Property vs Wire a Function

You can use `@wire` in two ways.

### Wire a property

```js
@wire(getRecord, {...})
account;
```

The wire service handles putting the results into `account.data` or `account.error`.

This is the **simpler approach** when you mainly need the data.

### Wire a function

```js
@wire(getRecord, {...})
wiredAccount({data, error}) {
    // custom logic
}
```

Use this when you need to **run logic whenever new data arrives**.

Because the wire service provides a **stream of values**, the function can execute multiple times. Therefore, the function should properly reset the states it controls:

```text
new data → data = value, error = undefined

error → error = value, data = undefined
```

### Mental distinction

> **Wire a property when you want the data.**
> **Wire a function when you want to do something when the data arrives.**

---

# 6. Modifying Data with LDS Functions

Wire adapters are mainly for **reading**.

To **create, update, or delete** records, use LDS functions.

For example:

```js
createRecord(recordInput)
```

Unlike `@wire`, this is **imperative**.

That means:

```text
User clicks button
       ↓
JavaScript handler runs
       ↓
createRecord(...)
       ↓
Salesforce operation
```

The function returns a **Promise**, so you handle success and failure with:

```js
.then(...)
.catch(...)
```

Also, LDS functions update the LDS cache, helping other components receive current data.

---

# 7. Important Limitation: Single Records and Transactions

The material makes an important distinction:

> LDS functions work with **single records**.

You can call multiple LDS functions, but each operation runs in its **own independent transaction**.

Therefore, there isn't a shared rollback across them.

For example:

```text
Create Account ✓
Create Contact ✗
```

You can't treat those two LDS calls as one transaction that automatically rolls back the Account creation.

If you need a **combined transactional DML operation**, the material says to consider **Apex**.

---

# The Big Picture

You can organize the choices like this:

```text
Need Salesforce data?
        │
        ├── Simple single-record form?
        │       ↓
        │   record-* forms
        │
        ├── Need to READ data with more control?
        │       ↓
        │   LDS wire adapter
        │
        └── Need to CREATE / UPDATE / DELETE?
                ↓
           LDS function
```

And the crucial distinction:

| Tool                    | Main job         | Style       |
| ----------------------- | ---------------- | ----------- |
| `lightning-record-form` | Simple record UI | Declarative |
| LDS wire adapter        | Read data        | Reactive    |
| LDS function            | Modify data      | Imperative  |

### Final mental model

**LDS is the data layer.**

* **Record forms** give you the easiest UI.
* **Wire adapters** let Salesforce **push/read data reactively**.
* **LDS functions** let your JavaScript **explicitly perform record changes**.
* **LDS cache** keeps data efficient and synchronized.

The progression is basically **simple → more control**, without immediately dragging Apex into the room.




# Use Apex to Work with Data

This unit answers an important question:

> **When should you stop using Lightning Data Service and use Apex instead?**

The answer is mainly about **complexity and control**. LDS is preferred when it fits, but Apex becomes the better choice when you need custom transactions or operations involving multiple records. 

## 1. When Apex Is Needed

LDS handles many common situations:

* simple record forms
* reading records
* creating, updating, or deleting a single record

But Apex is better when you need things such as:

* **custom single-record transactions**
* **multiple records in one transaction**
* data operations that aren't covered well by LDS

The big distinction is:

> **LDS is the simpler standard solution. Apex gives you more control.**

---

# 2. Making an Apex Method Available to LWC

An Apex method called from an LWC must be:

* `static`
* `public` or `global`
* annotated with `@AuraEnabled`

Example:

```apex
@AuraEnabled(cacheable=true)
public static List<Contact> getContactsBornAfter(Date birthDate) {
    ...
}
```

`@AuraEnabled` essentially tells Salesforce:

> "This Apex method can be called by Lightning components."

---

# 3. `cacheable=true`

For **read operations**, you can use:

```apex
@AuraEnabled(cacheable=true)
```

This allows the result to be cached, avoiding unnecessary server calls.

But there is an important restriction:

> **A cacheable Apex method cannot perform DML.**

So:

```text
cacheable=true → reading data
cacheable=true → ❌ creating/updating/deleting records
```

Also, because cached data can become stale, the material introduces `refreshApex` for refreshing data cached by Apex methods.

---

# 4. Two Ways to Call Apex

Just like with LDS, there are two approaches:

```text
             Apex
            /    \
        @wire   Imperative
```

## A. Call Apex with `@wire`

Use `@wire` when you want a **reactive read operation**.

```js
@wire(getContactsBornAfter, {
    birthDate: '$minBirthDate'
})
contacts;
```

The `$` makes `minBirthDate` reactive.

So:

```text
minBirthDate changes
        ↓
@wire reacts
        ↓
Apex runs
        ↓
contacts updated
```

Because wired Apex must be cacheable, this approach is for **reading**.

If you need to refresh the cached Apex result, use:

```js
refreshApex(...)
```

### Important LDS/Apex cache distinction

The material makes a subtle but important point:

> **LDS doesn't know about data cached by Apex methods.**

So if an LDS operation changes a record, an Apex method's cached result doesn't automatically know about that change. You may need `refreshApex`.

---

## B. Call Apex Imperatively

Imperative means **your JavaScript decides when Apex runs**.

```js
getContactsBornAfter({
    birthDate: this.minBirthDate
})
.then(contacts => {
    // success
})
.catch(error => {
    // failure
});
```

This is useful when:

* you want control over **when** the operation happens
* you're responding to a user action
* you're modifying records

Unlike wired Apex, imperative Apex can call both:

* cacheable methods
* non-cacheable methods

The result is a **Promise**, so you handle success with `.then()` and failure with `.catch()`.

### Easy distinction

> **`@wire` = reactive, framework-controlled.**
> **Imperative = explicitly invoked by your JavaScript.**

---

# 5. Apex + `lightning-datatable`

When you need to display a **list of records**, the recommended UI component in this material is:

`lightning-datatable`

It provides features such as:

* scrolling
* inline editing
* row actions
* resizing

A common architecture is:

```text
Apex
 ↓
@wire
 ↓
records.data
 ↓
lightning-datatable
 ↓
User sees table
```

The Apex method retrieves the records:

```apex
@AuraEnabled(cacheable=true)
public static List<Account> getAccounts() {
    return [
        SELECT Name, AnnualRevenue, Industry
        FROM Account
        WITH SECURITY_ENFORCED
        ORDER BY Name
    ];
}
```

Then the LWC wires to it:

```js
@wire(getAccounts)
accounts;
```

and the HTML supplies the data to the table:

```html
<lightning-datatable
    key-field="Id"
    data={accounts.data}
    columns={columns}>
</lightning-datatable>
```

So Apex handles **getting the records**, while `lightning-datatable` handles **displaying them**.

---

# 6. The Data-Access Decision Tree

This is probably the most useful part of the unit to memorize.

```text
Need Salesforce data
        ↓
Can LDS handle it?
        ↓
       YES
        ↓
       Use LDS
        │
        ├── Simple form → lightning-record-form
        ├── Custom view → record-view-form
        ├── Custom edit → record-edit-form
        ├── Read → LDS wire adapter
        └── Single-record change → LDS function
       
       NO
        ↓
      Apex
```

And for multiple records:

```text
Multiple records / complex transaction
              ↓
             Apex
```

---

# 7. Error Handling

The next unit connects directly to this one because **any server interaction can fail**.

The important thing is that the location of the error depends on how you called Salesforce.

### Wired property

```js
@wire(...)
accounts;
```

Error:

```js
accounts.error
```

### Wired function

```js
wiredAccounts({ data, error }) {
```

Error:

```js
error
```

### Imperative call

```js
someFunction()
    .then(...)
    .catch(error => {
        ...
    });
```

Error:

```js
catch(error)
```

The material uses `reduceErrors()` to turn Salesforce's error structures into easier-to-display messages. 

---

# Final Mental Model

Think of Salesforce data access as a **ladder of control**:

```text
Simplest
   ↓
Record Form
   ↓
LDS Wire Adapter
   ↓
LDS Function
   ↓
Apex
   ↓
Most customizable
```

The key rule is:

> **Use LDS when it can solve the problem. Use Apex when you need control that LDS doesn't provide.**

And for Apex itself:

> **`@wire` = reactive reads.**
> **Imperative Apex = you control when it runs, including record modifications.**
> **`cacheable=true` = cacheable reads, never DML.**

That distinction is the heart of this unit.
