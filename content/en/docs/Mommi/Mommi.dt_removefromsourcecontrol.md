# Technical Document for Stored Procedure: dbo.dt_removefromsourcecontrol

### Procedure Overview
The `dbo.dt_removefromsourcecontrol` stored procedure is designed to remove source control metadata from a database. This procedure specifically addresses the task of detaching a project or database object from version control by removing related properties from the `dtproperties` table. This can be particularly useful in a business scenario where a project is being retired or moved out of source control management.

### Input Parameters
This stored procedure does not accept any input parameters.

### Output Parameters
This stored procedure does not return any output parameters directly.

### Logic Description

1. **Set NOCOUNT ON**:
    - Ensures that the message indicating the number of rows affected by a SQL statement is not returned as a part of the result set.
  
    ```sql
    set nocount on
    ```

2. **Declare and Initialize Variables**:
    - Declares an integer variable `@iPropertyObjectId`.
    - Retrieves the `objectid` from the `dtproperties` table where the `property` is 'VCSProjectID' and stores it in `@iPropertyObjectId`.

    ```sql
    declare @iPropertyObjectId int
    select @iPropertyObjectId = (select objectid from dbo.dtproperties where property = 'VCSProjectID')
    ```

3. **Execute the Stored Procedure to Drop Properties**:
    - Calls the `dt_droppropertiesbyid` stored procedure, passing the `@iPropertyObjectId` as the first parameter and `null` as the second parameter. This step attempts to drop the properties associated with the fetched `objectid`.

    ```sql
    exec dbo.dt_droppropertiesbyid @iPropertyObjectId, null
    ```

4. **Error Handling**:
    - Checks the value of `@@error` immediately after the `exec` statement to determine if an error has occurred.
    - If `@@error` is neither `0` (success) nor `-1` (specific expected error code from `dt_droppropertiesbyid`), the procedure returns `1` to indicate failure.
    - If no error occurs or an expected error `-1` is encountered, the procedure proceeds to return `0` indicating success.

    ```sql
    if @@error <> 0 and @@error <> -1 return 1
    return 0
    ```

### Dependencies
- **Tables**:
  - `dtproperties`: Used to retrieve the `objectid` with the property 'VCSProjectID'.
  
- **Stored Procedures**:
  - `dbo.dt_droppropertiesbyid`: Called to drop properties by the specified `objectid`.

### Example Usage

```sql
-- Example SQL script to call dbo.dt_removefromsourcecontrol
BEGIN TRANSACTION

EXEC dbo.dt_removefromsourcecontrol

-- Check result of procedure
IF @@ROWCOUNT = 0
    PRINT 'Procedure executed successfully'
ELSE
    PRINT 'Procedure execution failed'

ROLLBACK TRANSACTION  -- remove this line and use COMMIT TRANSACTION to make changes permanent
```

### Error Handling
- **TRY-CATCH Blocks**:
  This procedure does not explicitly use TRY-CATCH blocks.

- **Error Codes**:
  Utilizes `@@error` to check for errors during the execution of the `dbo.dt_droppropertiesbyid` procedure. Returns `1` on unexpected errors and `0` on successful execution or expected outcomes.

### Performance Considerations
- The use of `SET NOCOUNT ON` helps to reduce the overhead associated with the default behavior of SQL Server sending messages about affected row counts.
- The stored procedure executes quickly, provided the `dtproperties` table is indexed appropriately for the `objectid` and `property` columns.

### Version History
(Optional)

| Date       | Description                                         |
|------------|-----------------------------------------------------|
| 2023-10-12 | Initial creation of the stored procedure documentation |

This concludes the detailed technical documentation for the `dbo.dt_removefromsourcecontrol` stored procedure. The document captures the key elements including functionality, input/output parameters, logic flow, dependencies, example usage, error handling, and performance considerations.