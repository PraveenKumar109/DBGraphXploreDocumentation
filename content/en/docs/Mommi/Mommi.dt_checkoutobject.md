# Technical Document for Stored Procedure `dbo.dt_checkoutobject`

## Procedure Overview
The stored procedure `dbo.dt_checkoutobject` is designed to interact with a version control system for SQL Server objects, primarily focused on stored procedures. The primary functions include checking out a stored procedure, retrieving its latest version, or undoing a checkout operation. By integrating with the version control system, this procedure helps maintain versioning and consistency, allowing multiple developers to work collaboratively on database objects.

## Input Parameters
The procedure accepts the following input parameters:

1. **@chObjectType** (char(4)): Specifies the type of SQL object. For this procedure, it expects the value 'PROC' indicating that the object is a stored procedure.
   - Expected Value: 'PROC'
   - Constraints: None

2. **@vchObjectName** (varchar(255)): The name of the SQL object (stored procedure) to be managed.
   - Expected Value: Name of an existing stored procedure (e.g., 'sp_ExampleProcedure')
   - Constraints: Must be a valid and existing stored procedure name.

3. **@vchComment** (varchar(255)): A comment that describes the reason for the checkout, the latest retrieval, or the undo operation.
   - Expected Value: Any descriptive text (e.g., 'Checking out for update')
   - Constraints: None

4. **@vchLoginName** (varchar(255)): The login name for accessing the version control system.
   - Expected Value: A valid login name (e.g., 'johndoe')
   - Constraints: Must correspond to a valid user in the version control system.

5. **@vchPassword** (varchar(255)): The password for the above login.
   - Expected Value: A valid password
   - Constraints: Must match the login credentials.

6. **@iVCSFlags** (int): Flags to control version control system behaviors.
   - Default Value: 0
   - Expected Value: 0 or system-specific flags
   - Constraints: As per version control system requirements

7. **@iActionFlag** (int): Determines the action to be performed - 0 for Checkout, 1 for Get Latest, 2 for Undo Checkout.
   - Default Value: 0
   - Expected Value: 0, 1, or 2
   - Constraints: Must be one of the defined values.

## Output Parameters
None.

## Logic Description
1. **Initialization:**
   - Various local variables are initialized, including IDs and GUIDs necessary for version control operations.

2. **Retrieve Version Control System Properties:**
   - The procedure uses `dt_getpropertiesbyid_vcs` to fetch necessary configuration properties like `VCSProject`, `VCSSourceSafeINI`, `VCSSQLServer`, and `VCSSQLDatabase` from the `dtproperties` table, which are essential for interacting with the version control system.

3. **Object Type Handling:**
   - The specific case of `@chObjectType` being 'PROC' (procedures) is handled, utilizing OLE Automation to interact with the version control system.

4. **Version Control System Interaction:**
   - Using `sp_OACreate` and `sp_OAMethod` to create an OLE object and call methods on it to perform checkout and other actions.
   - Errors encountered during these operations are handled using error-checking logic, which involves `sp_OAGetProperty` and `sp_OAError`.

5. **Streaming Data:**
   - The process involves fetching and inserting stream data into a temporary table `#commenttext` for temporary storage.
   - The procedure retrieves the stored procedure definition from `syscomments` for comparison or update purposes.

6. **Clean Up:**
   - Ensures cleanup of resources and exits the procedure gracefully even in the presence of errors.

## Dependencies
- **Tables:**
  - `dbo.dtproperties`: Used to fetch configuration properties for version control system integration.
  - `syscomments`: References the text of the stored procedure being checked out.

- **Stored Procedures:**
  - `dbo.dt_getpropertiesbyid_vcs`: Retrieves property values based on ID.
  - `dbo.dt_displayoaerror`: Handles and displays OLE Automation errors.

## Example Usage
The following SQL script demonstrates how to call the `dbo.dt_checkoutobject` procedure:

```sql
DECLARE @ReturnValue int;

EXEC @ReturnValue = dbo.dt_checkoutobject
    @chObjectType = 'PROC',
    @vchObjectName = 'sp_ExampleProcedure',
    @vchComment = 'Checking out for modification',
    @vchLoginName = 'johndoe',
    @vchPassword = 'securepassword',
    @iVCSFlags = 0,
    @iActionFlag = 0;

PRINT @ReturnValue;
```

## Error Handling
- The procedure uses conditional checks after each OLE Automation method call to determine if an error occurred.
- Errors trigger the `E_OAError` label, which calls the `dbo.dt_displayoaerror` procedure to log and handle the error.
- Execution then jumps to `CleanUp` to ensure resources are released properly.

## Performance Considerations
- **Use of Temporary Tables:** The procedure relies on a temporary table `#commenttext` for intermediate storage, which is efficient but may have performance implications under heavy load.
- **OLE Automation:** The use of OLE Automation can be resource-intensive and may affect performance. 
- **Index Utilization:** Ensure appropriate indexes on `dtproperties` and `syscomments` to optimize lookups.

## Version History
- **Initial Version:** Date of creation and initial implementation of the procedure. Further updates and changes should be logged here, including the date and a brief description.

---

End of Document.