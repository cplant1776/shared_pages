Lightning Message Service

# [Overview](https://developer.salesforce.com/docs/component-library/bundle/lightning-message-service/documentation)

Allows us to easily publish/subscribe to events from **anywhere** in the Lightning Experience. Useful for communicating with/between Aura/LWC components inside a Visualforce page.

## How to Use

1. Define it as a new metadata object:

> force-app/main/default/messageChannels/example.messageChannel-meta.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningMessageChannel xmlns="http://soap.sforce.com/2006/04/metadata">
    <masterLabel>SampleMessageChannel</masterLabel>
    <isExposed>true</isExposed>
    <description>This is a sample Lightning Message Channel.</description>
</LightningMessageChannel>
```

2. Subscribe/publish through it in your front-end

### LWC

> publish

```javascript
// publisherComponent.js
import { LightningElement, wire } from 'lwc';
import { publish, MessageContext } from 'lightning/messageService';
import SAMPLEMC from '@salesforce/messageChannel/SampleMessageChannel__c';

export default class PublisherComponent extends LightningElement {
    @wire(MessageContext)
    messageContext;
          
    handleClick() {
        const message = {
            recordId: '001xx000003NGSFAA4',
            recordData: {accountName: 'Burlington Textiles Corp of America'}
        };
        publish(this.messageContext, SAMPLEMC, message);
    }
}
```

> subscribe

```javascript
// subscribeComponent.js
import { LightningElement, wire } from 'lwc';
import { subscribe, unsubscribe, MessageContext } from 'lightning/messageService';

import SAMPLEMC from '@salesforce/messageChannel/SampleMessageChannel__c';

export default class SubscribeComponent extends LightningElement {
    @wire(MessageContext)
    messageContext;

    subscription = null;
    receivedMessage;

    subscribeMC() {
        if (this.subscription) {
            return;
        }
        this.subscription = subscribe(
            this.messageContext,
            SAMPLEMC, (message) => {
                this.handleMessage(message);
            });
    }

    unsubscribeMC() {
        unsubscribe(this.subscription);
        this.subscription = null;
    }

    handleMessage(message) {
        this.receivedMessage = message ? JSON.stringify(message, null, '\t') : 'no message payload';
    }
}
```

### Visualforce

> publish

```html
<apex:page>
    ...
    <script>
        // Load the Message Channel token in a variable
        var SAMPLEMC = "{!$MessageChannel.SampleMessageChannel__c}";
        function publishMC() {
            const message = {
                recordId: "some string",
                recordData: { value: "some value" } 
            }
            sforce.one.publish(SAMPLEMC, message);
        }
    </script>
    ...
</apex:page>
```

> (un)subscribe

```html
<apex:page>
    ...
    <script>
        // Load the Message Channel token in a variable
        var SAMPLEMC = "{!$MessageChannel.SampleMessageChannel__c}";
        var subscription;
 
        function handleMessage(message) {
            var textArea = document.querySelector("#messageTextArea");
            textArea.innerHTML = message ? JSON.stringify(message, null, '\t') : 'no message payload';
        }
 
        function subscribeMC() {
            if (!subscription) {
                subscription = sforce.one.subscribe(SAMPLEMC, handleMessage);
            }
        }
 
        function unsubscribeMC() {
            if (subscription) {
                sforce.one.unsubscribe(subscription);
                subscription = null;
            }
        }
    </script>
    ...
</apex:page>
```

### Aura

> publish

```html
<!-- myPublisherComponent.cmp -->
<aura:component>
    <lightning:button onclick="{!c.publishMC}">
    <lightning:messageChannel type="SampleMessageChannel__c"
                              aura:id="sampleMessageChannel"/>
</aura:component>
```


```javascript
// myPublisherComponentController.js
({
    publishMC: function(cmp, event, helper) {
        var message = {
            recordId: "some string",
            recordData: { value: "some value" }
        };
        cmp.find("sampleMessageChannel").publish(message);
    }
})
```

> subscribe

```html
<!-- mySubscriberComponent.cmp -->
<aura:component>
    <aura:attribute name="recordValue" type="String"/>
    <lightning:formattedText value="{!v.recordValue}" />
    <lightning:messageChannel type="SampleMessageChannel__c"
                              onMessage="{!c.handleMessage}"/>
</aura:component>
```

```javascript
// mySubscriberComponentController.js
({
    handleMessage: function(cmp, message, helper) { 
        // Read the message argument to get the values in the message payload
        if (message != null && message.getParam("recordData") != null) {
            cmp.set("v.recordValue", message.getParam("recordData").value);
        }
    }
})
```