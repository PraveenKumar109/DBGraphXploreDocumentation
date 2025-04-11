# Technical Document for `dbo.dt_getpropertiesbyid` Stored Procedure

## Procedure Overview

**Procedure Name:** `dbo.dt_getpropertiesbyid`  
**Purpose:** The purpose of this stored procedure is to retrieve properties related to an object identified by `id` from the `dbo.dtproperties` table. It can return either all properties for a given object or a specific property if specified.

**Business Use Case:** This procedure is useful in scenarios where metadata or properties associated with objects need to be fetched. This can include data for auditing, reporting, or enhancing the functionality of applications that rely on object metadata.

## Input Parameters

1. **@id**
   - **Data Type:** `int`
   - **Purpose:** This parameter uniquely identifies the object whose properties are to be retrieved.
   - **Expected Values:** An integer value representing a valid object ID present in the `dbo.dtproperties` table.
   - **Constraints:** Must be a valid integer corresponding to an existing object in the system.

2. **@property**
   - **Data Type:** `varchar(64)`
   - **Purpose:** This parameter specifies the property name to be retrieved for the given object.
   - **Expected Values:** A string (up to 64 characters) representing the name of the property.
   - **Constraints:** Can be null or an empty string to retrieve all properties for the specified object.

## Output Parameters

This stored procedure does not utilize any output parameters. The results are returned as a result set.

## Logic Description

1. **Set NOCOUNT On:** `set nocount on` is used at the beginning to prevent the message indicating the number of rows affected by a T-SQL statement from being displayed.
   
2. **Parameter Check & Conditional Logic:**
   - If `@property` is null or an empty string:
     - A `SELECT` query is executed to retrieve all properties (`property`, `version`, `value`, `lvalue`) for the object with the specified `@id`.
   - If `@property` is not null or non-empty:
     - A `SELECT` query is executed to retrieve the specified property (`property`, `version`, `value`, `lvalue`) for the object with the specified `@id`.

## Dependencies

- **Table - `dbo.dtproperties`:** This table holds the properties, versions, values, and long values associated with objects. The procedure queries this table to fetch the required property information.

## Example Usage

Here is an example SQL script demonstrating how to call the `dbo.dt_getpropertiesbyid` procedure:

```sql
-- Example: Retrieve all properties for the object with ID 5
EXEC dbo.dt_getpropertiesbyid @id = 5, @property = NULL;

-- Example: Retrieve a specific property 'PropertyName' for the object with ID 5
EXEC dbo.dt_getpropertiesbyid @id = 5, @property = 'PropertyName';
```

## Error Handling

The stored procedure currently does not include explicit error handling mechanisms such as TRY-CATCH blocks or specific error codes. It relies on SQL Server's default error handling behavior. For improved robustness, consider adding error handling to manage potential issues such as invalid `@id` or database connectivity problems.

## Performance Considerations

- **Indexes:** Ensure that the `dbo.dtproperties` table has appropriate indexes on the `objectid` and `property` columns to optimize the query performance.
- **Parameter Value Checks:** Input parameters are checked minimally; more rigorous validation could further enhance efficiency and reliability.

## Version History

(Optional, if applicable)

- **Initial Version:**
  - **Date:** [Date]
  - **Description:** Initial creation of the stored procedure to retrieve properties by `id` and optionally by `property`.

## Conclusion

The `dbo.dt_getpropertiesbyid` stored procedure provides a straightforward method for retrieving object properties from the `dbo.dtproperties` table based on an object identifier and optional property name. It helps in efficiently querying and managing object metadata within the database, thereby supporting various business applications.