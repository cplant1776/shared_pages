Salesforce Data in LWC

# Overview

There's multiple ways to access/modify Salesforce data from Lightning.
1. Lightning Data Service
2. Wire Adapters
3. Apex Methods

## [Lightning Data Service (SLDS)](https://developer.salesforce.com/docs/component-library/bundle/lightning-record-form/documentation)

### Overview

Simplist and prefferred way. When multiple components utilize it, a change in data is reflected across all components.

#### lightning-record-*-form

Easiest way to work with single records.

##### Types

1. ```lightning-record-form```
Most basic. Just specify a layout and the fields displayed are based on the page layout selected. Can override and manually define fields too
2. ```lightning-record-view-form```
3. ```lightning-record-edit-form```
View and edit forms let us prepoulate values and customize form layout

##### Example

Using a record form to create an account
> accountCreator.html
```html
<template>
    <lightning-card>
        <lightning-record-form
            object-api-name={objectApiName}
            fields={fields}
            onsuccess={handleSuccess}>
        </lightning-record-form>
    </lightning-card>
</template>
```
> accountCreator.js
```javascript
import { LightningElement } from 'lwc';
import { ShowToastEvent } from 'lightning/platformShowToastEvent';
import ACCOUNT_OBJECT from '@salesforce/schema/Account';
import NAME_FIELD from '@salesforce/schema/Account.Name';
import REVENUE_FIELD from '@salesforce/schema/Account.AnnualRevenue';
import INDUSTRY_FIELD from '@salesforce/schema/Account.Industry';
export default class AccountCreator extends LightningElement {
    objectApiName = ACCOUNT_OBJECT;
    fields = [NAME_FIELD, REVENUE_FIELD, INDUSTRY_FIELD];
    handleSuccess(event) {
        const toastEvent = new ShowToastEvent({
            title: "Account created",
            message: "Record ID: " + event.detail.id,
            variant: "success"
        });
        this.dispatchEvent(toastEvent);
    }
}
```
* Note how the fields are imported in the JS.
* "A Toast is a non modal, unobtrusive window element used to display brief, auto-expiring windows of information to a user"

## [Wire Adapters/Functions](https://developer.salesforce.com/docs/component-library/documentation/en/lwc/lwc.reference_ui_api)

### Read

Use @wire
> On a Property

```javascript
import { LightningElement, api, wire } from 'lwc';
import { getRecord } from 'lightning/uiRecordApi';
import ACCOUNT_NAME_FIELD from '@salesforce/schema/Account.Name';
export default class WireGetRecordProperty extends LightningElement {
    @api recordId;
    @wire(getRecord, { recordId: '$recordId', fields: [ACCOUNT_NAME_FIELD] })
    account;
}
```
* ```@api recordId``` passes Id of current record to the component
* Retrieved data accessed with ```account.data```
* ```$``` prefix on ```$recordId``` makes record reactive for all components

> On a function

```javascript
import { LightningElement, api, wire } from 'lwc';
import { getRecord } from 'lightning/uiRecordApi';
import ACCOUNT_NAME_FIELD from '@salesforce/schema/Account.Name';
export default class WireGetRecord extends LightningElement {
    @api recordId;
    data;
    error;
    @wire(getRecord, { recordId: '$recordId', fields: [ACCOUNT_NAME_FIELD] })
    wiredAccount({data, error}) {
        console.log('Execute logic each time a new value is provisioned');
        if (data) {
            this.data = data;
            this.error = undefined;
        } else if (error) {
            this.error = error;
            this.data = undefined;
        }
    }
}
```

### Update

Use "LDS functions"
```javascript
import { LightningElement} from 'lwc';
import { createRecord } from 'lightning/uiRecordApi';
import ACCOUNT_OBJECT from '@salesforce/schema/Account';
import ACCOUNT_NAME_FIELD from '@salesforce/schema/Account.Name';
export default class LdsCreateRecord extends LightningElement {
    handleButtonClick() {
        const recordInput = {
            apiName: ACCOUNT_OBJECT.objectApiName,
            fields: {
                [ACCOUNT_NAME_FIELD.fieldApiName] : 'ACME'
            }
        };
        createRecord(recordInput)
            .then(account => {
                // code to execute if create operation is successful
            })
            .catch(error => {
                // code to execute if create operation is not successful
            });
    }
}
```
* Invokes the LDS createRecord() function inside a handle function
* Only works on a single record at a time

### Useful Docs
[uiRecord API](https://developer.salesforce.com/docs/component-library/documentation/en/lwc/lwc.reference_lightning_ui_api_record)
[uiObjectInfo API](https://developer.salesforce.com/docs/component-library/documentation/en/lwc/lwc.reference_lightning_ui_api_object_info)

## [Apex Methods](https://developer.salesforce.com/docs/component-library/documentation/en/lwc/apex)

### Overview

Most flexibility. Operate on multiple records in a single transaction.


#### Expose the Method
```Apex
public with sharing class AccountController {
    @AuraEnabled(cacheable=true)
    public static List<Contact> getRelatedContacts(Id searchId) {
        return [
            SELECT Name, Title, Email, Phone
            FROM Contact
            WHERE AccountId = :searchId
            WITH SECURITY_ENFORCED
       ];
    }
}
```
* ```@AuraEnabled``` lets Aura and Lightning Components
* ```(cacheable=true)``` utilizes the cache, but makes DML unusable

##### Call Apex with @wire

MUST BE CACHABLE to use @wire. Call ```refreshApex()``` if data might have been updated on Apex side.

```javascript
import { LightningElement, api, wire } from 'lwc';
import getRelatedContacts from '@salesforce/apex/AccountController.getRelatedContacts';
export default class WireApexProperty extends LightningElement {
    @api recordId;
    @wire(getRelatedContacts, { accountId: '$recordId' })
    contacts;
}
```

##### Call Apex Imperatively

```javascript
import { LightningElement, api, wire } from 'lwc';
import getRelatedContacts from '@salesforce/apex/AccountController.getRelatedContacts';
export default class CallApexImperative extends LightningElement {
    @api recordId;
    handleButtonClick() {
        getRelatedContacts({ //imperative Apex call
            accountId: '$recordId'
        })
            .then(contacts => {
                //code to execute if related contacts are returned successfully
            })
            .catch(error => {
                //code to execute if related contacts are not returned successfully
            });
    }
}
```

#### Example

Creating a list of Accounts in a \<lightning-datatable>

> AccountController

```Apex
public with sharing class AccountController {
    @AuraEnabled(cacheable=true)
    public static List<Account> getAccounts() {
        return [
            SELECT Name, AnnualRevenue, Industry
            FROM Account
            WITH SECURITY_ENFORCED
            ORDER BY Name
        ];
    }
}
```

> accountList.html

```html
<template>
    <lightning-card>
        <template if:true={accounts.data}>
            <lightning-datatable
                key-field="Id"
                data={accounts.data}
                columns={columns}
            >
           </lightning-datatable>
        </template>
    </lightning-card>
</template>
```
> accountList.js

```javascript
import { LightningElement, wire } from 'lwc';
import NAME_FIELD from '@salesforce/schema/Account.Name';
import REVENUE_FIELD from '@salesforce/schema/Account.AnnualRevenue';
import INDUSTRY_FIELD from '@salesforce/schema/Account.Industry';
import getAccounts from '@salesforce/apex/AccountController.getAccounts';
const COLUMNS = [
    { label: 'Account Name', fieldName: NAME_FIELD.fieldApiName, type: 'text' },
    { label: 'Annual Revenue', fieldName: REVENUE_FIELD.fieldApiName, type: 'currency' },
    { label: 'Industry', fieldName: INDUSTRY_FIELD.fieldApiName, type: 'text' }
];
export default class AccountList extends LightningElement {
    columns = COLUMNS;
    @wire(getAccounts)
    accounts;
}
```

## Error Handling

### Wired Properties

```javascript
import { LightningElement, api, wire } from 'lwc';
import { reduceErrors } from 'c/ldsUtils';
import getRelatedContacts from '@salesforce/apex/AccountController.getRelatedContacts';
export default class WireApexProperty extends LightningElement {
    @api recordId;
    @wire(getRelatedContacts, { accountId: '$recordId' })
    contacts;
    get errors() {
        return (this.contacts.error) ?
            reduceErrors(this.contacts.error) : [];
    }
}
```

### Wired Functions

```javascript
import { LightningElement, api, wire } from 'lwc';
import { reduceErrors } from 'c/ldsUtils';
import getRelatedContacts from '@salesforce/apex/AccountController.getRelatedContacts';
export default class WireApexFunction extends LightningElement {
    @api recordId;
    errors;
    @wire(getRelatedContacts, { accountId: '$recordId' })
    wiredContacts({data, error}) {
        if (error)
            this.errors = reduceErrors(error);
    }
}
```

### Imperative Function Calls

```javascript
import { LightningElement, api, wire } from 'lwc';
import { reduceErrors } from 'c/ldsUtils';
import getRelatedContacts from '@salesforce/apex/AccountController.getRelatedContacts';
export default class CallApexImperative extends LightningElement {
    @api recordId;
    errors;
    handleButtonClick() {
        getRelatedContacts({
            accountId: '$recordId'
        })
            .then(contacts => {
                // code to execute if the promise is resolved
            })
            .catch(error => {
                this.errors = reduceErrors(error); // code to execute if the promise is rejected
            });
    }
}
```