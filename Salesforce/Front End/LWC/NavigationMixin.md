NavigationMixin

# [Overview](https://developer.salesforce.com/docs/component-library/bundle/lightning-navigation/documentation)

Used to navigate to pages in your org within LWC.

## Using

### Importing

Note that we have to extend **NavigationMixin**

```javascript
import { NavigationMixin } from 'lightning/navigation';
export default class MyCustomElement extends NavigationMixin(LightningElement) {}
```

### Navigating

```javascript
import { LightningElement } from 'lwc';
import { NavigationMixin } from 'lightning/navigation';

export default class Example extends NavigationMixin(LightningElement) {
    navigateToObjectHome() {
        // Navigate to the Account home page
        this[NavigationMixin.Navigate]({
            type: 'standard__objectPage',
            attributes: {
                objectApiName: 'Account',
                actionName: 'home',
            },
        });
    }

    recordPageUrl;

    connectedCallback() {
        // Generate a URL to a User record page
        this[NavigationMixin.GenerateUrl]({
            type: 'standard__recordPage',
            attributes: {
                recordId: '005B0000001ptf1IAE',
                actionName: 'view',
            },
        }).then(url => {
            this.recordPageUrl = url;
        });
    }
}
```

- **[NavigationMixin.Navigate](pageReference, [replace])** - navigates to another page in the App
- **\[NavigationMixin.GenerateUrl](pageReference)** - A component calls this API to get a promise that resolves to the resulting URL. The component can use the URL in the href attribute of an anchor. It can also use the URL to open a new window using the window.open(url) browser API.