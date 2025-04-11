# Technical Documentation for SQL Stored Procedure `dbo.dt_validateloginparams`

## Procedure Overview
The `dbo.dt_validateloginparams` stored procedure is designed to validate user login parameters against a version control system. The procedure takes a login name and password as inputs and verifies them using the `ValidateLoginParams` method of an OLE Automation object. This stored procedure is particularly useful in scenarios where access to a system or application is controlled via a version control system like Visual SourceSafe.

## Input Parameters
1. **@vchLoginName**
   - **Data Type**: `varchar(255)`
   - **Purpose**: Represents the user's login name.
   - **Expected Values**: A string containing the login name of the user. There are no specific constraints other than the length limit.

2. **@vchPassword**
   - **Data Type**: `varchar(255)`
   - **Purpose**: Represents the user's password.
   - **Expected Values**: A string containing the password of the user. There are no specific constraints other than the length limit.

## Output Parameters
There are no explicit output parameters defined for this stored procedure.

## Logic Description
The stored procedure follows these steps:

1. **Initialization**:
   - The `set nocount on` statement is used to prevent the sending of DONE_IN_PROC messages to the client for each statement in a stored procedure.
   - Declare integer and varchar variables needed for subsequent operations (`@iReturn`, `@iObjectId`, `@VSSGUID`, `@iPropertyObjectId`, `@vchSourceSafeINI`).

2. **Initialization of Variables**:
   - `@iObjectId` is initialized to 0.
   - `@VSSGUID` is set to a hard-coded value representing the GUID of the Visual SourceSafe OLE Automation object.

3. **Retrieve Object ID**:
   - `@iPropertyObjectId` is set by selecting the `objectid` from `dbo.dtproperties` where the `property` is 'VCSProjectID'.

4. **Retrieve SourceSafeINI Property**:
   - Execute the stored procedure `dbo.dt_getpropertiesbyid_vcs` to get the `VCSSourceSafeINI` property value and store it in `@vchSourceSafeINI`.

5. **Create OLE Automation Object**:
   - Use `master.dbo.sp_OACreate` to create an instance of the OLE Automation object with the GUID stored in `@VSSGUID`. The returned status is stored in `@iReturn`. If the return status is non-zero, an error-handling routine (`E_OAError`) is invoked.

6. **Validate Login Parameters**:
   - Call the `ValidateLoginParams` method of the created OLE Automation object using `master.dbo.sp_OAMethod`. Pass the `@vchSourceSafeINI`, `@vchLoginName`, and `@vchPassword` as parameters. The returned status is stored in `@iReturn`. If the return status is non-zero, the error-handling routine (`E_OAError`) is invoked.

7. **Cleanup**:
   - The procedure returns, concluding its operations.

8. **Error Handling**:
   - An error-handling routine named `E_OAError` is defined. If an error occurs, the stored procedure `dbo.dt_displayoaerror` is called with the `@iObjectId` and `@iReturn` values to display the error, followed by cleanup and exit.

## Dependencies
- Tables:
  - `dbo.dtproperties`: This table is queried to find the `objectid` based on the `property` name 'VCSProjectID'.
- Stored Procedures:
  - `dbo.dt_getpropertiesbyid_vcs`: Called to retrieve the `VCSSourceSafeINI` property value.
  - `master.dbo.sp_OACreate`: Used to create an instance of the OLE Automation object.
  - `master.dbo.sp_OAMethod`: Used to call a method on the OLE Automation object.
  - `dbo.dt_displayoaerror`: Used to display OLE Automation errors.

## Example Usage
```sql
DECLARE @loginName varchar(255) = 'testUser'
DECLARE @password varchar(255) = 'testPassword'

EXEC dbo.dt_validateloginparams @vchLoginName = @loginName, @vchPassword = @password;
```

## Error Handling
Errors within the stored procedure are handled via the `E_OAError` label. When an error occurs during the creation of the OLE Automation object or the invocation of its methods, control is transferred to the `E_OAError` section:
- The procedure `dbo.dt_displayoaerror` is called to display the error information.
- The stored procedure then proceeds to the `CleanUp` section to gracefully exit.

## Performance Considerations
- **OLE Automation**: The use of OLE Automation procedures, such as `sp_OACreate` and `sp_OAMethod`, can introduce overhead and potential performance bottlenecks. Best practices should be followed to ensure these are used judiciously.
- **Database Queries**: Ensure the table `dbo.dtproperties` is indexed properly for the `property` column to optimize performance of the select query.

## Version History
None provided in the initial definition. Document the version history as changes are made to the procedure.

---

This comprehensive documentation should provide a clear understanding of the `dbo.dt_validateloginparams` stored procedure's functionality, usage, and components.