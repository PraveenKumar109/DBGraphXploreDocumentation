# Technical Document: `dbo.dt_getobjwithprop_u` Stored Procedure

## Procedure Overview

The stored procedure `dbo.dt_getobjwithprop_u` is designed to retrieve the object(s) associated with a specified property in the `dbo.dtproperties` table. This procedure is essential for scenarios where it is necessary to find out which object has a given property, potentially with a specific value.

### Key Functionality:
- Retrieves object IDs based on a given property name.
- Allows for an optional value (uvalue) to narrow down the search when specified.
- Facilitates data consistency and retrieval efficiency in scenarios related to object-property associations.

### Business Use Case:
This procedure is useful in database management and audit scenarios where understanding the link between objects and properties in the `dbo.dtproperties` table is crucial. It can help in analyzing metadata, verifying configurations, and tracking property allocations to objects.

## Input Parameters

| Parameter Name | Data Type     | Purpose                                               | Expected Values / Constraints                 |
| -------------- | ------------- | ----------------------------------------------------- | --------------------------------------------- |
| `@property`    | `varchar(30)` | Name of the property to search for.                  | Must be a non-null, non-empty string. Maximum length is 30 characters.|
| `@uvalue`      | `nvarchar(255)` | Optional value associated with the property to narrow the search. | Can be null. If provided, maximum length is 255 characters. |

## Output Parameters

This stored procedure does not have any output parameters.

## Logic Description

### Overview:
The procedure performs its function primarily through conditional checking and a SELECT query. It checks the validity of the `@property` parameter and retrieves `objectid`s based on the given parameters.

### Step-by-Step Explanation:
1. **Disable Result Set Count Output:**
   ```sql
   set nocount on;
   ```
   This prevents the sending of DONE_IN_PROC messages to the client after each statement, which can enhance performance.

2. **Validation of `@property` parameter:**
   ```sql
   if (@property is null) or (@property = '')
   begin
       raiserror('Must specify a property name.', -1, -1);
       return (1);
   end;
   ```
   - Checks whether the `@property` parameter is either null or an empty string.
   - If invalid, raises an error with the message 'Must specify a property name.' and exits with a return code of 1.

3. **Conditional Object Retrieval Based on `@uvalue`:**
   - When `@uvalue` is null:
     ```sql
     select objectid id from dbo.dtproperties where property=@property;
     ```
     Retrieves object IDs where the property matches `@property`.

   - When `@uvalue` is not null:
     ```sql
     select objectid id from dbo.dtproperties where property=@property and uvalue=@uvalue;
     ```
     Retrieves object IDs where the property matches `@property` and the uvalue matches `@uvalue`.

## Dependencies

- **Tables:**
  - `dbo.dtproperties`: This table is used to store object properties and the corresponding values. The stored procedure references the table to fetch object IDs based on provided criteria.

## Example Usage

### Sample SQL Script:

```sql
-- Example 1: Retrieve objects with a specific property
EXEC dbo.dt_getobjwithprop_u @property = 'ExampleProperty', @uvalue = null;

-- Example 2: Retrieve objects with a specific property and value
EXEC dbo.dt_getobjwithprop_u @property = 'ExampleProperty', @uvalue = 'ExampleValue';
```

### Expected Results:
- **Example 1:** Returns object IDs where the property is 'ExampleProperty', disregarding the value.
- **Example 2:** Returns object IDs where the property is 'ExampleProperty' and the value is 'ExampleValue'.

## Error Handling

- The procedure uses the `RAISERROR` function to handle errors:
  ```sql
  if (@property is null) or (@property = '')
  begin
      raiserror('Must specify a property name.', -1, -1);
      return (1);
  end;
  ```
  - If `@property` is null or an empty string, an error message is raised and the procedure exits with a return code of 1.

## Performance Considerations

- **Use of `SET NOCOUNT ON`:** This improves performance by suppressing the message that indicates the number of rows affected by a T-SQL statement.
- **Indexes on `dbo.dtproperties`:** Ensure that there are appropriate indexes on the `property` and `uvalue` columns in `dbo.dtproperties` to optimize query performance.
- **Conditional Logic:** Minimal and efficient operations are included ensuring the procedure runs quickly if parameters are specified correctly.

## Version History

| Date       | Description               | Version |
|------------|---------------------------|---------|
| YYYY-MM-DD | Initial creation          | 1.0     |
| YYYY-MM-DD | Added error handling      | 1.1     |
| YYYY-MM-DD | Performance optimization  | 1.2     |

(Note: Replace `YYYY-MM-DD` with actual date of changes)

---

This document provides a comprehensive overview of the stored procedure `dbo.dt_getobjwithprop_u`, detailing its functionality, parameters, internal logic, dependencies, usage examples, error handling mechanisms, and performance considerations.