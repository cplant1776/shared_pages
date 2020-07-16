Toast Event

# [Overview](https://developer.salesforce.com/docs/component-library/bundle/lightning-platform-show-toast-event/documentation)

Pops up a little message to the user informing them of something.

## Example

```JavaScript
import { LightningElement } from 'lwc';
import { ShowToastEvent } from 'lightning/platformShowToastEvent'
export default class MyComponent extends LightningElement {
    showToast() {
        const event = new ShowToastEvent({
            title: 'Get Help',
            message: 'Salesforce documentation is available in the app. Click ? in the upper-right corner.',
        });
        this.dispatchEvent(event);
    }
}
```