## Technical Document for Stored Procedure `dbo.dt_getobjwithprop`

### Procedure Overview
The stored procedure `dbo.dt_getobjwithprop` is designed to retrieve the owner object(s) based on a specified property and optionally, its corresponding value from the `dtproperties` table. This procedure is useful for querying metadata properties and their associated objects within a database. It can be employed in scenarios where system administrators or developers need to identify objects based on specific characteristics or custom properties stored in the database.

### Input Parameters
#### 1. @property
- **Data Type**: `varchar(30)`
- **Purpose**: Specifies the name of the property for which the owner objects are being queried.
- **Expected Values**: The name should be a valid property name present in the `dtproperties` table. It cannot be `NULL` or an empty string (`''`).

#### 2. @value
- **Data Type**: `varchar(255)`
- **Purpose**: Specifies the value of the property for which the owner objects are being queried. 
- **Expected Values**: This parameter can be `NULL`. If provided, it should be a valid property value present in the `dtproperties` table.

### Output Parameters
This stored procedure does not use output parameters. The result set of the procedure consists of a single column:
- **id**: The `objectid` from the `dtproperties` table representing the ID of the owner object.

### Logic Description
1. **Set No Count**: 
   - `set nocount on` prevents the message indicating the number of rows affected by a SQL statement from being returned, which is purely for performance tuning and cleaner output.

2. **Validation for `@property`**:
   - The procedure checks if the `@property` parameter is `NULL` or an empty string. If true, it raises an error 'Must specify a property name.' and returns with a status code `1`.

3. **Query Execution**:
   - If `@value` is `NULL`, it performs a SELECT query on `dtproperties` looking for rows where the `property` column matches `@property` and returns the `objectid` column as `id`.
   - If `@value` is not `NULL`, it performs a SELECT query on `dtproperties` looking for rows where both `property` matches `@property` and `value` matches `@value`, returning the `objectid` column as `id`.

### Dependencies
- **Tables**:
  - `dbo.dtproperties`: The procedure queries this table to retrieve the `objectid` based on the specified property and value.
  
- **Views/Other Procedures**: None.

### Example Usage
```sql
-- Example 1: Retrieve objects with a specific property name
EXEC dbo.dt_getobjwithprop @property = 'CreatedBy', @value = NULL;

-- Example 2: Retrieve objects with a specific property name and value
EXEC dbo.dt_getobjwithprop @property = 'CreatedBy', @value = 'admin';
```

### Error Handling
- The procedure uses a simple validation mechanism to ensure that the `@property` parameter is not `NULL` or empty. If validation fails, it raises an error with the message 'Must specify a property name.’ and exits with a return code `1`.
- There is no TRY-CATCH block implemented in this procedure, so any other errors during execution will propagate to the calling environment.

### Performance Considerations
- The `set nocount on` statement is used to optimize performance by preventing the sending of row count messages after each statement.
- Indexes on the `property` and `value` columns in the `dtproperties` table can significantly improve the performance of the SELECT queries, especially for large datasets.
- The procedure’s performance depends on the efficiency of queries against the `dtproperties` table; ensuring the table is well-maintained and indexed is critical for optimal performance.

### Version History
(Optional - Not included as there are no previous version details provided)

---

This technical document provides a comprehensive overview and detailed explanation of the stored procedure `dbo.dt_getobjwithprop`. It is designed to assist developers and database administrators in understanding, using, and potentially modifying the stored procedure for their specific needs.