# Technical Document for Stored Procedure [dbo].[mui_AncillaryProductSupplier_lst]

## Procedure Overview
The stored procedure `[dbo].[mui_AncillaryProductSupplier_lst]` is designed to handle specific operations related to listing ancillary product suppliers. This procedure ensures that the system is not in maintenance mode before proceeding with any operations and validates the connection ID. It then retrieves the appropriate procedure call from the redirector table and executes it. This is critical for ensuring that the application runs correctly and securely, particularly in environments where multiple versions of procedures might exist and system maintenance may occur.

## Input Parameters
This section details the input parameters accepted by the procedure:

1. **@ConnectionID**
   - **Data Type**: `uniqueidentifier`
   - **Purpose**: Used to identify the user's connection for validation purposes.
   - **Expected Values or Constraints**: Should be a valid globally unique identifier (GUID). It can be null, in which case the procedure skips certain validation steps.

2. **@VersionNumber**
   - **Data Type**: `int`
   - **Purpose**: Indicates the version number of the procedure call to be used.
   - **Expected Values or Constraints**: Default value is 1. It should be a positive integer.

## Output Parameters
There are no explicit output parameters for this stored procedure. However, the procedure does return an integer, which serves as a status code. The following status codes are returned:
- `0`: Success
- `-9`: Error, either due to system maintenance or an invalid connection ID.

## Logic Description
The stored procedure performs the following steps:

1. **Initialization**: Sets procedure-specific variables like `@ReturnCode`, `@ProcedureCall`, `@ProcedureName`, and `@SubroutineNumber`.

2. **Maintenance Mode Check**:
   - Calls the `[dbo].[SystemConfiguration_Info]` procedure to check if the system is in maintenance mode.
   - If the system is in maintenance mode (i.e., `@AccessDenied = 'Y'`), it raises error 50003 and returns `-9`.

3. **Connection ID Validation**:
   - If `@ConnectionID` is provided, the procedure validates it by calling `[sr_ConnectionUserAssoc]`.
   - If the validation fails (i.e., `@ReturnCode != 1`), it raises error 50008 and returns `-9`.

4. **Retrieve Procedure Call**:
   - Uses the User-Defined Function (UDF) `dbo.udf_ProcedureCall` to get the actual procedure call string based on the procedure name, version number, and subroutine number.
   - If the procedure call retrieval fails (i.e., `@ProcedureCall IS NULL`), it raises error 50001 and returns `-9`.

5. **Execute the Procedure Call**:
   - Executes the retrieved procedure call and sets the return code accordingly.

6. **Return**:
   - Returns the `@ReturnCode` indicating the success or failure of the procedure.

## Dependencies
This stored procedure references several database objects:

1. **Procedures**:
   - `[dbo].[SystemConfiguration_Info]`: Checks if the system is in maintenance mode.
   - `[sr_ConnectionUserAssoc]`: Validates the connection ID.

2. **User-Defined Functions**:
   - `dbo.udf_ProcedureCall`: Retrieves the procedure call string based on the provided parameters.

## Example Usage

Here is an example of how to call the `[dbo].[mui_AncillaryProductSupplier_lst]` procedure:

```sql
DECLARE @ReturnCode int;

EXEC @ReturnCode = [dbo].[mui_AncillaryProductSupplier_lst]
    @ConnectionID = 'A67E1309-B332-4F25-B5A9-9A65EE4C9C8F',
    @VersionNumber = 1;

IF @ReturnCode = 0
    PRINT 'Procedure executed successfully.';
ELSE
    PRINT 'Procedure failed with return code: ' + CAST(@ReturnCode as varchar);
```

## Error Handling
Errors in this procedure are handled using the `RAISERROR` function with specific error codes:

1. **50003**: Raised when the system is in maintenance mode.
2. **50008**: Raised when the provided connection ID is invalid.
3. **50001**: Raised when the procedure call string cannot be retrieved.

The procedure returns `-9` for all error conditions to signal failure.

## Performance Considerations
- **SET NOCOUNT ON**: This command is used to stop the message indicating the number of rows affected by a T-SQL statement, which can help improve performance by reducing unnecessary network traffic.
- **Efficient Validation**: The procedure uses specific validation steps to ensure minimal overhead.
- **Deferred Execution**: By dynamically retrieving and executing the procedure call, it allows more flexibility and optimization based on version numbers and specific subroutine calls.

## Version History
- **Version 4**: Updated by Iochman on 12/13/04, as indicated by the header comment. Further history can be maintained by updating this section with relevant changes.

This detailed documentation covers the various aspects of the stored procedure `[dbo].[mui_AncillaryProductSupplier_lst]`, providing insights into its purpose, functionality, and usage.