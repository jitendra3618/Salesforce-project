### Types of Triggers

* Before triggers - are used to update or validate record values before they are saved to the database.
* After triggers - are used to access field values that are set by the system (such as record's Id or `LastModifiedDate` field), and to affect changes in other records.  The records that fire the after trigger are read-only.

### Using Context Variables
Triggers are often used to access and manage records related to the records in the trigger context - the records that caused this trigger to fire.

To access the records that caused the trigger to fire, use context variables. 
* `Trigger.New` contains all the records that were inserted in `insert` or `update` triggers. 
* `Trigger.Old` provides the old version of sObjects before they were updated in update triggers, or a list of deleted sObjects in delete triggers. 

Triggers can fire when one record is inserted, or when many records are inserted in bulk via the API or Apex. Therefore, context variables, such as `Trigger.New`, can contain only one record or multiple records. You can iterate over `Trigger.New` to get each individual sObject.

| Variable | Usage |
| --- | --- |
| isExecuting | Returns true if the current context for the Apex code is a trigger, not a Visualforce page, a Web service, or an executeanonymous() API call. |
| isInsert | Returns true if this trigger was fired due to an insert operation, from the Salesforce user interface, Apex, or the API. |
| isUpdate | Returns true if this trigger was fired due to an update operation, from the Salesforce user interface, Apex, or the API. |
| isDelete | Returns true if this trigger was fired due to a delete operation, from the Salesforce user interface, Apex, or the API. | 
| isBefore | Returns true if this trigger was fired before any record was saved. |
| isAfter | Returns true if this trigger was fired after all records were saved. |
| isUndelete | Returns true if this trigger was fired after a record is recovered from the Recycle Bin. This recovery can occur after an undelete operation from the Salesforce user interface, Apex, or the API. |
| new | Returns a list of the new versions of the sObject records.  This sObject list is only available in insert, update, and undelete triggers, and the records can only be modified in before triggers. |
| newMap | A map of IDs to the new versions of the sObject records.  This map is only available in before update, after insert, after update, and after undelete triggers. |
| old | Returns a list of the old versions of the sObject records.  This sObject list is only available in update and delete triggers. |
| oldMap | A map of IDs to the old versions of the sObject records.  This map is only available in update and delete triggers. |
| operationType | Returns an enum of type System.TriggerOperation corresponding to the current operation.  Possible values of the System.TriggerOperation enum are: BEFORE_INSERT, BEFORE_UPDATE, BEFORE_DELETE,AFTER_INSERT, AFTER_UPDATE, AFTER_DELETE, and AFTER_UNDELETE. If you vary your programming logic based on different trigger types, consider using the switch statement with different permutations of unique trigger execution enum states. |
| size | The total number of records in a trigger invocation, both old and new. |

### Using Trigger Exceptions
To prevent saving records in a trigger, call the `addError()` method on the sObject in question.  The `addError()` method throws a fatal error inside a trigger.  The error message is displayed in the user interface and is logged.

Calling addError() in a trigger causes the entire set of operations to roll back, except when bulk DML is called with partial success.
* If a DML statement in Apex spawned the trigger, any error rolls back the entire operation. However, the runtime engine still processes every record in the operation to compile a comprehensive list of errors.
* If a bulk DML call in the Lightning Platform API spawned the trigger, the runtime engine sets the bad records aside. The runtime engine then attempts a partial save of the records that did not generate errors.

### Triggers and Callouts
To make a callout from a trigger, call a class method that executes asynchronously.  Such a method is called a future method and is annotated with `@future(callout=true)`.  

### Performing Bulk SOQL
Query limits are:
* 100 SOQL queries for synchronous Apex or 
* 200 for asynchronous Apex.

Best practices:
* The SOQL query uses an inner query—(SELECT Id FROM Opportunities)—to get related opportunities of accounts.
* The SOQL query is connected to the trigger context records by using the IN clause and binding the Trigger.New variable in the WHERE clause—WHERE Id IN :Trigger.New. This WHERE condition filters the accounts to only those records that fired this trigger.

Triggers execute on batches of 200 records at a time. So if 400 records cause a trigger to fire, the trigger fires twice, once for each 200 records. For this reason, you don’t get the benefit of SOQL for loop record batching in triggers, because triggers batch up records as well. The SOQL for loop is called twice in this example, but a standalone SOQL query would also be called twice. However, the SOQL for loop still looks more elegant than iterating over a collection variable!

### Performing Bulk DML
The Apex runtime allows up to 150 DML calls in one transaction.
