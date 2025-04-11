# Technical Document for Stored Procedure `dbo.dt_checkinobject`

## Procedure Overview
The stored procedure `dbo.dt_checkinobject` is designed to manage the check-in and addition of objects, specifically SQL stored procedures, to a version control system (VCS) from within the SQL Server environment. This enables proper tracking and versioning of changes made to stored procedures. The procedure handles both "AddFile" and "CheckIn" actions, ensuring that the stored procedures can be managed efficiently within the VCS.

## Input Parameters

| Parameter Name   | Data Type    | Purpose                                                                                                                                                     | Expected Values or Constraints                               |
|------------------|--------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------|
| `@chObjectType`  | `char(4)`    | Specifies the type of object. Currently supports 'PROC' for stored procedures.                                                                               | 'PROC'                                                       |
| `@vchObjectName` | `varchar(255)` | The name of the object (e.g., stored procedure) being managed in the VCS.                                                                                     | Name of an existing stored procedure                         |
| `@vchComment`    | `varchar(255)` | An optional comment describing the changes being made during the check-in or add file action. Defaults to an empty string.                                   | Free text                                                    |
| `@vchLoginName`  | `varchar(255)` | The username used to authenticate with the VCS.                                                                                                                  | Valid VCS login name                                         |
| `@vchPassword`   | `varchar(255)` | The password associated with the VCS login name. Defaults to an empty string.                                                                                  | Valid VCS password                                           |
| `@iVCSFlags`     | `int`          | Flags for additional options or configurations specific to the VCS.                                                                                            | Integer value                                                |
| `@iActionFlag`   | `int`          | Determines the action to be performed: 0 for "AddFile" and 1 for "CheckIn".                                                                                     | 0 (AddFile) or 1 (CheckIn)                                   |
| `@txStream1`     | `Text`         | The SQL script for the drop stream.                                                                                                                              | Valid SQL drop stream text. Defaults to an empty string.     |
| `@txStream2`     | `Text`         | The SQL script for the create stream.                                                                                                                            | Valid SQL create stream text. Defaults to an empty string.   |
| `@txStream3`     | `Text`         | The SQL script for the grant stream.                                                                                                                             | Valid SQL grant stream text. Defaults to an empty string.    |

## Output Parameters
There are no output parameters for this stored procedure.

## Logic Description
The logic flow of the `dbo.dt_checkinobject` stored procedure comprises the following key steps:

1. **Initialization**:
    - Sets `NOCOUNT ON`.
    - Declares and initializes necessary local variables such as `@iObjectId`, `@iStreamObjectId`, `@VSSGUID`, `@iPropertyObjectId`, `@vchProjectName`, `@vchSourceSafeINI`, `@vchServerName`, `@vchDatabaseName`, `@iReturnValue`, `@pos`, and `@vchProcLinePiece`.

2. **Retrieve VCS Configuration**:
    - Retrieves VCS-related properties from the `dbo.dtproperties` table.

3. **Stream Compilation**:
    - If `@chObjectType` is 'PROC' and `@iActionFlag` is 1 (CheckIn):
        - Begins a transaction.
        - Attempts to compile the SQL streams (`@txStream1`, `@txStream2`, `@txStream3`).

4. **Object Automation (OLE Automation)**:
    - Create and obtain the VCS object using OLE Automation procedures (`sp_OACreate` and `sp_OAGetProperty`).
  
5. **Handle Stream Addition**:
    - For CheckIn action (`@iActionFlag` = 1):
        - Splits the `@txStream2` into smaller pieces and inserts them into the VCS using `sp_OAMethod`.
        - Finalizes the check-in operation by invoking the `CheckIn_StoredProcedure` method.
    - For AddFile action (`@iActionFlag` = 0):
        - Retrieves the procedure lines, splits them, and adds them to the VCS using `sp_OAMethod`.

6. **Transaction Management**:
    - Commits the transaction if the compilation succeeds and the action flag is 1.
    - Rolls back the transaction if compilation fails or an OLE Automation error occurs.

7. **Error Handling**:
    - Jumps to error-handling labels and executes the necessary cleanup and error-reporting steps.

## Dependencies
The `dbo.dt_checkinobject` stored procedure relies on several database objects:

- **Tables**:
  - `dbo.dtproperties`: Retrieves VCS configuration properties.
  
- **Other Stored Procedures**:
  - `dbo.dt_getpropertiesbyid_vcs`: Retrieves VCS property values by ID.
  - `master.dbo.sp_OACreate`: Creates an instance of an OLE Automation object.
  - `master.dbo.sp_OAGetProperty`: Gets a property value from an OLE Automation object.
  - `master.dbo.sp_OAMethod`: Calls a method on an OLE Automation object.
  - `dbo.dt_displayoaerror`: Displays the error from an OLE Automation operation.

## Example Usage
```sql
-- Example call to add a file to the VCS
EXEC dbo.dt_checkinobject
    @chObjectType = 'PROC',
    @vchObjectName = 'usp_myStoredProcedure',
    @vchComment = 'Initial check-in of stored procedure',
    @vchLoginName = 'myVCSUser',
    @vchPassword = 'myVCSPassword',
    @iVCSFlags = 0,
    @iActionFlag = 0,
    @txStream1 = 'DROP PROCEDURE usp_myStoredProcedure;',
    @txStream2 = 'CREATE PROCEDURE usp_myStoredProcedure AS BEGIN SELECT 1 END;',
    @txStream3 = 'GRANT EXECUTE ON usp_to myRole'

-- Example call to check-in changes to the VCS
EXEC dbo.dt_checkinobject
    @chObjectType = 'PROC',
    @vchObjectName = 'usp_myStoredProcedure',
    @vchComment = 'Updated logic to enhance performance',
    @vchLoginName = 'myVCSUser',
    @vchPassword = 'myVCSPassword',
    @iVCSFlags = 0,
    @iActionFlag = 1,
    @txStream1 = 'DROP PROCEDURE usp_myStoredProcedure;',
    @txStream2 = 'CREATE PROCEDURE usp_myStoredProcedure AS BEGIN SELECT 1 END;',
    @txStream3 = 'GRANT EXECUTE ON usp_to myRole'
```

## Error Handling
- The procedure includes error-handling logic to manage errors effectively:
  - **Compilation Errors**:
    - Checked using `@@ERROR` after executing the SQL streams.
    - If an error is detected, the transaction is rolled back, and the error is raised using `RAISEERROR`.
    
  - **OLE Automation Errors**:
    - Checks the return values of OLE Automation methods.
    - On error, rolls back the transaction (if applicable) and calls `dbo.dt_displayoaerror` to display detailed error information.

## Performance Considerations
- **Transaction Handling**:
  - Transactions are used to ensure that multiple related operations are treated as a single unit, improving data consistency.
  
- **Stream Processing**:
  - Large text streams are split into smaller chunks (255 characters) and processed in loops to mitigate potential performance issues related to handling large amounts of data.

- **Error Management**:
  - Structured error handling with `TRY-CATCH` equivalent logic ensures minimal impact on performance while providing robust feedback and rollback capabilities.

## Version History
*(Optional: Include any updates or changes to the procedure)*

---

This document provides a comprehensive overview of the `dbo.dt_checkinobject` stored procedure, detailing its purpose, functionality, parameters, logic, dependencies, example usage, error handling, and performance considerations. This should aid in understanding and maintaining the procedure effectively.