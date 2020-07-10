Apex

## Example Object

<details>
	<summary>Show Details</summary>

### Name

Support_Tier

### Fields

* Minimum_Spending__c
* Default_Discount__c

</details>

## Variable Declaration

Use CustomMedataTypeName **__mdt**

```apex
Support_Tier__mdt Bronze;
<CustomMetdataType>__mdt <VarName>;
```

## Getting Field Values

Note the use of **__c** in the SOQL

```apex
Support_Tier__mdt [] minSpent = [SELECT Minimum_Spending__c FROM Support_Tier__mdt];
```

## Editing Custom Metadata in Apex

We don't use DML to save changes to the database. We use ```Metadata.DeployContainer```

### Example

```apex
public class CustomMetadataService {
    public CustomMetadataService() {}
    /**
     * This method instantiates a custom metadata record of type Support_Tier__mdt
     * and sets the DeveloperName to the input String.
     * The record is not inserted into the database, 
     * and would not be found by a SOQL query.
     */
    public Support_Tier__mdt getCustomMetadataRecord(String myName) {
        Support_Tier__mdt supportTier = new Support_Tier__mdt();
        theRecord.DeveloperName = myName;
        return supportTier;
    }
    /**
     * This method retrieves a custom metadata record, changes a field, and returns it
     * to the caller, but does not update the database.
     */
    public Support_Tier__mdt getChangedCustomMetadataRecord(String myNewName) {
        Support_Tier__mdt supportTier = [SELECT Id, DeveloperName from Support_Tier__mdt LIMIT 1];
        theRecord.DeveloperName = myNewName;
        return supportTier;
    }
}

```