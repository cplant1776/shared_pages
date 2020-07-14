Metadata API

# Overview

Lets us make (limited) changes to an Org's metadata programatically. Example: adding a new field to a page layout with a script. Can be used to create Setup Wizards to guide admins through a configuration task.

## Supported Changes (so far)

* Page layouts
* Records of custom metadata types
* **No destructive changes**


## Example

### Add Custom Field to Account Layout

<details>
	<summary>Show Details</summary>

#### Classes

* **UpdatePageLayout** - Actually updates the layout
* **PostInstallCallback** - Callback function to run after completion of deployment (implements **Metadata.DeployCallback**)
* **DeployMetadata** - Container that queues the deployment (uses **Metadata.Operations.enqueueDeployment**)
* **PostInstallScript** - Deploys metadata after install (implements **InstallHandler**)

> UpdatePageLayout

```apex
public class UpdatePageLayout {
    // Add custom field to page layout
    
    public Metadata.Layout buildLayout() {
        
        // Retrieve Account layout and section 
        List<Metadata.Metadata> layouts = 
            Metadata.Operations.retrieve(Metadata.MetadataType.Layout, 
            new List<String> {'Account-Account Layout'});
        Metadata.Layout layoutMd = (Metadata.Layout) layouts.get(0);
        Metadata.LayoutSection layoutSectionToEdit = null;
        List<Metadata.LayoutSection> layoutSections = layoutMd.layoutSections;
        for (Metadata.LayoutSection section : layoutSections) {
            
            if (section.label == 'Account Information') {
                layoutSectionToEdit = section;
                break;
            }
        }
        
        // Add the field under Account info section in the left column
        List<Metadata.LayoutColumn> layoutColumns = layoutSectionToEdit.layoutColumns;     
        List<Metadata.LayoutItem> layoutItems = layoutColumns.get(0).layoutItems;
        
        // Create a new layout item for the custom field
        Metadata.LayoutItem item = new Metadata.LayoutItem();
        item.behavior = Metadata.UiBehavior.Edit;
        item.field = 'AMAPI__Apex_MD_API_sample_field__c';
        layoutItems.add(item);
        
        return layoutMd;
    }
}

```

> PostInstallCallback

```apex
public class PostInstallCallback implements Metadata.DeployCallback {
  
    public void handleResult(Metadata.DeployResult result,
        Metadata.DeployCallbackContext context) {
        
        if (result.status == Metadata.DeployStatus.Succeeded) {
            // Deployment was successful, take appropriate action.
            System.debug('Deployment Succeeded!');
        } else {
            // Deployment wasn’t successful, take appropriate action.
	    System.debug('Deployment Failed!');
        }
    }
}
```

> DeployMetadata

```apex
public class DeployMetadata {
 
    // Create metadata container 
    public Metadata.DeployContainer constructDeploymentRequest() {
        
        Metadata.DeployContainer container = new Metadata.DeployContainer();
        
        // Add components to container         
        Metadata.Layout layoutMetadata = new UpdatePageLayout().buildLayout();
        container.addMetadata(layoutMetadata);
        return container;
    }
    
    // Deploy metadata
    public void deploy(Metadata.DeployContainer container) {
        // Create callback. 
        PostInstallCallback callback = new PostInstallCallback();
        
        // Deploy the container with the new components. 
        Id asyncResultId = Metadata.Operations.enqueueDeployment(container, callback);
    }
}
```

> PostInstallScript

```apex
public class PostInstallScript implements InstallHandler {
    
    // Deploy post-install metadata  
    public void onInstall(InstallContext context) {
        DeployMetadata deployUtil = new DeployMetadata();
        Metadata.DeployContainer container = deployUtil.constructDeploymentRequest();
        deployUtil.deploy(container);
    }
}
```

</details>

### Add Custom Metadata Through Visualforce

<details>
	<summary>Show Details</summary>

#### Scenario

We want to create a front-end for admins to easily enter a new type of metadata (VAT Rates). We just create a basic visualforce front-end and an Apex controller that utilizes the Metadata API

> VATController

```apex
public class VATController {
    
    public final List<VAT_Rate__mdt> VATs {get;set;}
    final Map<String, VAT_Rate__mdt> VATsByApiName {get; set;}
    
    public VATController() { 
        VATs = new List<VAT_Rate__mdt>();
        VATsByApiName = new Map<String, Vat_Rate__mdt>();
        for (VAT_Rate__mdt v : [SELECT QualifiedApiName, MasterLabel, Default__c, Rate__c
                                FROM VAT_Rate__mdt]) { 
                                    VATs.add(v);
                                    VATsByApiName.put(v.QualifiedApiName, v);
                                }
    }
    
    public PageReference save() {        
        
        // Create a metadata container.
        Metadata.DeployContainer container = new Metadata.DeployContainer();
        List<String> vatFullNames = new List<String>();
        for (String recordName : VATsByApiName.keySet()) {
            vatFullNames.add('VAT_Rate.' + recordName);
        }
        
        List<Metadata.Metadata> records = 
            Metadata.Operations.retrieve(Metadata.MetadataType.CustomMetadata, 
                                         vatFullNames);
        
        for (Metadata.Metadata record : records) {
            Metadata.CustomMetadata vatRecord = (Metadata.CustomMetadata) record;
            String vatRecordName = vatRecord.fullName.substringAfter('.');
            VAT_Rate__mdt vatToCopy = VATsByApiName.get(vatRecordName);
            
            for (Metadata.CustomMetadataValue vatRecValue : vatRecord.values) {
                vatRecValue.value = vatToCopy.get(vatRecValue.field);
            }
            
            // Add record to the container.
            container.addMetadata(vatRecord);
        }   
        
        // Deploy the container with the new components. 
        Id asyncResultId = Metadata.Operations.enqueueDeployment(container, null);
        
        return null;
    }
}
```

> VATRateForm

```html
<apex:page controller="VATController">
    <apex:form >
        <apex:pageBlock title="VAT Rates" mode="edit">
            <apex:pageMessages />
            
            <apex:pageBlockButtons >
                <apex:commandButton action="{!save}" value="Save"/>
            </apex:pageBlockButtons>
            
            <apex:pageBlockTable value="{!VATs}" var="v">
                <apex:column value="{!v.MasterLabel}"/>
                <apex:column headerValue="Rate">
                    <apex:inputText value="{!v.Rate__c}"/>
                </apex:column>
                <apex:column headerValue="Default">
                    <apex:inputCheckbox value="{!v.Default__c}"/>
                </apex:column>
            </apex:pageBlockTable>
        </apex:pageBlock>
    </apex:form>
</apex:page>
```

</details>

## Testing

<details>
	<summary>Show Details</summary>

Test the Deployment Container

> DeploymentTest

```apex
@IsTest
public class DeploymentTest {
    @IsTest
    static void testDeployment() {
        DeployMetadata deployMd = new DeployMetadata();
        
        Metadata.DeployContainer container = deployMd.constructDeploymentRequest();
        List<Metadata.Metadata> contents = container.getMetadata();
        System.assertEquals(1, contents.size());
        Metadata.Layout md = (Metadata.Layout) contents.get(0);
       
        // Perform various assertions the layout metadata.
        System.assertEquals('Account-Account Layout', md.fullName);
    }
}
```

Test the Callback

```apex
@IsTest
public class MyDeploymentCallbackTest {
    @IsTest
    static void testMyCallback() {
        
        // Instantiate the callback.
        Metadata.DeployCallback callback = new PostInstallCallback();
       
        // Create test result and context objects.
        Metadata.DeployResult result = new Metadata.DeployResult();
        result.numberComponentErrors = 1;
        Metadata.DeployCallbackContext context = new Metadata.DeployCallbackContext();
        
        // Invoke the callback's handleResult method.
        callback.handleResult(result, context);
    }
}
```

**note:** If a non-null Id value is needed for the callback, create a subclass extending **Metadata.DeployCallbackContext**

```apex
// DeployCallbackContext subclass for testing that returns myJobId.
public class TestingDeployCallbackContext extends Metadata.DeployCallbackContext {
    private Id myJobId = null; // Set to a fixed ID you can use for testing.
    public override Id getCallbackJobId() {
        return myJobId;
    }
}
```

</summary>
