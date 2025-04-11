# Technical Document for Stored Procedure: `dbo.dt_adduserobject_vcs`

## Procedure Overview
The `dbo.dt_adduserobject_vcs` stored procedure is designed to facilitate the addition of a new user-defined property to the `dbo.dtproperties` table within a SQL Server database. Its primary function is to check if a property with the given name already exists in the table. If it does not exist, it creates a new entry. This procedure is particularly useful in configuration management and ensuring unique property entries without duplicates.

## Input Parameters
### `@vchProperty`
- **Data Type**: `varchar(64)`
- **Purpose**: The name of the property to be added to the `dbo.dtproperties` table.
- **Expected Values/Constraints**: A string with a maximum length of 64 characters. It should be unique within the `property` column of the `dbo.dtproperties` table.

## Output Parameters
There are no explicit output parameters. However, the stored procedure returns an integer value which is the `objectid` of the inserted or existing property.

## Logic Description
1. **Set NOCOUNT ON**: This command stops the message that shows the number of rows affected by SQL statements, which helps in reducing network traffic.
2. **Declare `@iReturn`**: An integer variable to store the `objectid`.
3. **Begin Transaction**: Initiates a transaction to ensure atomicity.
4. **Select `objectid`**: Query the `dbo.dtproperties` table to check if a property with the given `@vchProperty` name already exists.
   - If the property exists (`@iReturn` is not NULL), the transaction is committed and the existing `objectid` is returned.
   - If the property does not exist (`@iReturn` is NULL):
     - Insert the new property into the `dbo.dtproperties` table.
     - Update the `objectid` for the new row using the `@@identity` value which captures the identity value of the last inserted row.
     - Retrieve the `objectid` of the newly inserted property.
5. **Commit Transaction**: Ends the transaction and makes all changes permanent.
6. **Return `@iReturn`**: Returns the `objectid` of the processed property.

## Dependencies
### Tables
- **`dbo.dtproperties`**: The table where properties are stored. This table must exist in the database for the procedure to function correctly.

### Views
- None

### Other Stored Procedures
- None

## Example Usage
```sql
-- Example of calling the stored procedure
DECLARE @newObjectId INT;
EXEC @newObjectId = dbo.dt_adduserobject_vcs @vchProperty = 'NewPropertyName';
PRINT 'The Object ID is ' + CAST(@newObjectId as VARCHAR);
```
- **Input**: `@vchProperty = 'NewPropertyName'`
- **Expected Output**: The `objectid` of the newly created or existing property.

## Error Handling
This stored procedure does not explicitly include structured error handling mechanisms like TRY-CATCH blocks. Potential error scenarios include:
- Violations of constraints on the `dbo.dtproperties` table.
- Database connection issues.
- Transaction conflicts that might require a rollback.

Adding structured error handling could enhance the robustness of this procedure.

## Performance Considerations
- **Transaction Scope**: The transaction ensures the atomicity of the insert and update operations. Ensure that these operations are optimized and the table is indexed appropriately to avoid locking issues.
- **Indexing**: Ensure the `property` column in the `dbo.dtproperties` table is indexed to improve performance on lookups.
- **Set NOCOUNT**: Using `SET NOCOUNT ON` reduces the overhead of network traffic.

## Version History
- **Initial Version**: 
  - **Date**: October 2023
  - **Description**: Created the stored procedure `dbo.dt_adduserobject_vcs` to add a new user object property if it does not already exist.

This document provides a full overview of the `dbo.dt_adduserobject_vcs` stored procedure, covering its purpose, usage, and technical aspects to ensure proper usage and maintenance.