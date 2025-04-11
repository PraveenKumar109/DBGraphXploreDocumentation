## Technical Document for Stored Procedure `dbo.dt_adduserobject`

### Procedure Overview

The stored procedure `dbo.dt_adduserobject` is used to add a new entry to the `dtproperties` table in the database. Specifically, it inserts a property named 'DtgSchemaOBJECT' and updates the `objectid` field of the newly inserted record with the identity value generated upon insertion. This is typically used for managing schema objects or tracking specific properties in a controlled manner.

### Input Parameters

This stored procedure does not take any input parameters.

### Output Parameters

This stored procedure does not define any explicit output parameters. However, it returns the identity value (`@@identity`) of the newly inserted record as its return value which can be captured by the calling process. This return value is of type `int`.

### Logic Description

The logic of the stored procedure is structured as follows:

1. **Disable row count messaging**: 
    ```sql
    set nocount on
    ```
   This command is used to prevent the message that shows the number of rows affected by the SQL statements from being returned to the client.

2. **Begin a transaction**:
    ```sql
    begin transaction
    ```
   This ensures that the operations are atomic; either all operations succeed, or none do, maintaining the integrity of the data.

3. **Insert a new record into `dtproperties`**:
    ```sql
    insert dbo.dtproperties (property) VALUES ('DtgSchemaOBJECT')
    ```
   A new entry with the property 'DtgSchemaOBJECT' is inserted into the `dtproperties` table.

4. **Update the newly inserted record's `objectid`**:
    ```sql
    update dbo.dtproperties set objectid=@@identity 
        where id=@@identity and property='DtgSchemaOBJECT'
    ```
   The `objectid` of the record just inserted is set to the value of its own identity (ID) column. This links the `objectid` to its unique identifier.

5. **Commit the transaction**:
    ```sql
    commit
    ```

6. **Return the identity value**:
    ```sql
    return @@identity
    ```
   The stored procedure returns the identity value of the newly inserted record.

### Dependencies

- **Tables:**
  - `dtproperties`: This table must exist in the database. It must have at least the columns `id`, `objectid`, and `property`.

### Example Usage

Here's an example of how you might call this stored procedure and capture the returned identity value:

```sql
DECLARE @NewId int;
EXEC @NewId = dbo.dt_adduserobject;
PRINT 'Newly inserted object ID is: ' + CAST(@NewId AS varchar);
```

### Error Handling

The stored procedure does not include explicit error handling mechanisms such as `TRY-CATCH` blocks. If an error occurs (e.g., a constraint violation), the transaction will be automatically rolled back by SQL Server. For better robustness, consider adding error handling, for example:

```sql
BEGIN TRY
    begin transaction
        insert dbo.dtproperties (property) VALUES ('DtgSchemaOBJECT')
        update dbo.dtproperties set objectid=@@identity 
            where id=@@identity and property='DtgSchemaOBJECT'
    commit
    return @@identity
END TRY
BEGIN CATCH
    IF @@TRANCOUNT > 0
        ROLLBACK TRANSACTION;
    THROW;
END CATCH
```

### Performance Considerations

- The procedure wraps operations in a transaction, which is good for maintaining data integrity but might impact performance slightly due to transactional overhead.
- The use of `@@identity` limits the procedure to the scope of the current session. Ensure no other commands modifying identities are run concurrently in the same session.
- The `set nocount on` command helps reduce unnecessary network traffic, slightly improving performance.

### Version History

- **Initial Version**:
  - **Date**: 2023-10-01
  - **Description**: Creation of the stored procedure to add a user-defined object to the `dtproperties` table and return the newly generated identity value.

---

This document has been prepared to provide a comprehensive overview of the `dbo.dt_adduserobject` stored procedure, ensuring clarity in its purpose, functionality, and implementation for developers and database administrators.