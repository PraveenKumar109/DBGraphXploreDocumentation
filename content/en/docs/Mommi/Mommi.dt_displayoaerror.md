# Technical Document for Stored Procedure: `dbo.dt_displayoaerror`

## Procedure Overview
The stored procedure `dbo.dt_displayoaerror` is designed to fetch and display error information for a given OLE automation object. The procedure calls a system stored procedure to retrieve error details and raises a descriptive error message. This is particularly useful in scenarios where detailed error handling and output for OLE automation processes are required, such as integrating SQL Server with external applications via COM objects.

## Input Parameters
- **@iObject** (int): 
    - **Purpose**: The identifier of the OLE automation object for which the error information needs to be retrieved.
    - **Expected Values**: Valid integer value representing a handle to an OLE object.

- **@iresult** (int):
    - **Purpose**: Represents the result code of the previous OLE automation operation. Typically used to determine if the operation was successful or if there was an error.
    - **Expected Values**: Valid integer value, usually representing the result/status of an OLE automation operation.

## Output Parameters
This stored procedure does not have any output parameters.

## Logic Description
1. **Set NOCOUNT ON**: This statement is used to disable the count of rows affected by Transact-SQL statements from being returned, which can improve performance by reducing network traffic.
2. **Variable Declarations**: Several local variables are declared:
    - `@vchOutput` (varchar(255)): A string to hold the combined error source and description.
    - `@hr` (int): Variable to store the result of the system procedure call.
    - `@vchSource` (varchar(255)): A string to receive the source of the error information.
    - `@vchDescription` (varchar(255)): A string to receive the description of the error.
3. **Calling System Procedure**: `master.dbo.sp_OAGetErrorInfo` is executed to fetch the error details associated with the OLE automation object identified by `@iObject`. The source and description of the error are stored in `@vchSource` and `@vchDescription` respectively.
4. **Combining Error Information**: The error source and description are concatenated into the `@vchOutput` variable.
5. **Raising Error**: The combined error message is raised as an SQL Server error using `RAISERROR`, specifying a severity level of 16 and a state of -1.
6. **Return**: The procedure ends execution and returns control to the caller.

## Dependencies
- **System Stored Procedure**: `master.dbo.sp_OAGetErrorInfo`
    - **Role**: Retrieves the error information for the specified OLE automation object.

## Example Usage
```sql
DECLARE @iObject int
DECLARE @iresult int

-- Assuming @iObject has been set up previously
SET @iObject = 12345

-- Example result indicating an error
SET @iresult = -1

-- Call the stored procedure
EXEC dbo.dt_displayoaerror @iObject, @iresult
```
In this example, replace `12345` with the actual OLE automation object handle.

## Error Handling
- **Raise Error**: The procedure ensures that any error information fetched from the OLE automation process is raised as an SQL Server error. The `RAISERROR` function is used to generate an error with a message created by concatenating the error source and description.
- There is no explicit `TRY-CATCH` block in this procedure, but the procedure relies on the system stored procedure `sp_OAGetErrorInfo` to obtain error information.

## Performance Considerations
- **Set NOCOUNT ON**: By setting NOCOUNT ON, the procedure avoids sending unnecessary row count messages to the client, which can reduce overhead.
- **String Handling**: The procedure combines string values to create an error message, but since the strings are relatively short, this has minimal performance impact.
- **Procedure Calls**: The `sp_OAGetErrorInfo` system procedure call can introduce overhead depending on that procedure's implementation and the specific OLE object being used.

## Version History
No version history provided. 

---

This document outlines the stored procedure `dbo.dt_displayoaerror` in detail, covering its purpose, input parameters, logic flow, dependencies, example usage, error handling, and performance considerations.