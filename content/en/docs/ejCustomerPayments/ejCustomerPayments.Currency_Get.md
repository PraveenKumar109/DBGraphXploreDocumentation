---
title: Currency_Get
weight: 20
---
## Procedure Overview

The purpose of the stored procedure `[dbo].[Currency_Get]` is to retrieve details about different currencies from the `Currency` table. This includes fetching essential attributes such as the currency's ID, code, description, symbol, activity status, and the number of significant decimal places. This procedure is primarily used in the context of a payment processing service within a broader payment system.

### Business Use Case

In the context of a payment processing service, various currencies need to be validated and potentially displayed or used in transactions. The stored procedure `[dbo].[Currency_Get]` facilitates these operations by providing a streamlined way to fetch all necessary currency information from the database.

## Input Parameters

This stored procedure does not require any input parameters.

## Output Parameters

This stored procedure does not return any output parameters directly. Instead, it returns a result set containing the following columns:
- **CurrencyId** (Id): INTEGER - The unique identifier for each currency.
- **Code**: NVARCHAR(50) - The alphanumeric code representing the currency (e.g., USD, EUR).
- **Description**: NVARCHAR(255) - A short description of the currency.
- **Symbol**: NVARCHAR(10) - The symbol associated with the currency (e.g., $, €, ¥).
- **IsActive**: BIT - A flag indicating whether the currency is active (1) or inactive (0).
- **SignificantDecimalPlaces**: INT - The number of significant decimal places used in currency transactions.

## Logic Description

Here's a step-by-step explanation of the procedure's internal logic:

1. **Variable Declaration**:
   ```sql
   DECLARE @ProcedureContext VARBINARY(128)
   ```

   The procedure declares a variable called `@ProcedureContext` of type `VARBINARY(128)`.

2. **SET NOCOUNT ON**:
   ```sql
   SET NOCOUNT ON
   ```

   This command prevents the sending of DONE_IN_PROC messages to the client for each statement in a stored procedure, which can improve performance slightly by reducing network traffic.

3. **Setting Context Information**:
   ```sql
   SET @ProcedureContext = CAST('PaymentService_Payment-Processing' AS VARBINARY(128))
   SET CONTEXT_INFO  @ProcedureContext
   ```

   The `@ProcedureContext` variable is assigned a binary value representing the string 'PaymentService_Payment-Processing'. This context info can be used for various purposes such as tracking the stored procedure's usage or debugging.

4. **Selecting Currency Details**:
   ```sql
   SELECT CurrencyId Id
       ,Code
       ,[Description]
       ,Symbol
       ,IsActive
       ,SignificantDecimalPlaces
   FROM Currency
   ```

   The main logic of the procedure is a SELECT statement that retrieves currency details from the `Currency` table. This result set includes:
   - `CurrencyId` (aliased as `Id`)
   - `Code`
   - `Description`
   - `Symbol`
   - `IsActive`
   - `SignificantDecimalPlaces`

## Dependencies

The stored procedure `[dbo].[Currency_Get]` references the `Currency` table. Below is an explanation of the table's role:

- **Currency Table**:
  - Contains core information about various currencies.
  - Each row represents a unique currency with attributes such as its code, description, symbol, active status, and significant decimal places.

## Example Usage

```sql
-- Example of calling the stored procedure
EXEC [dbo].[Currency_Get]

-- Expected Result Set
-- | Id  | Code | Description | Symbol | IsActive | SignificantDecimalPlaces |
-- | --- | ---- | ----------- | ------ | -------- | ----------------------- |
-- | 1   | USD  | US Dollar   | $      | 1        | 2                       |
-- | 2   | EUR  | Euro        | €      | 1        | 2                       |
-- ... (Additional rows)
```

## Error Handling

The procedure does not include explicit error handling mechanisms such as TRY-CATCH blocks. Given its current scope, which involves a straightforward SELECT statement, error handling might be unnecessary. However, it can be augmented with error handling for more robust applications.

## Performance Considerations

### Optimization Techniques Used
- **SET NOCOUNT ON**: The use of this command can help reduce overhead by minimizing the number of messages sent to the client.
- **Efficient Query**: The SELECT statement is straightforward and leverages the inherent indexing of the primary key in the `Currency` table.

### Potential Bottlenecks
- **Large Dataset**: If the `Currency` table grows significantly, the SELECT query might have performance implications. Consider indexing common query fields or using pagination for large result sets.

### Recommendations
- Regularly monitor query performance and adjust indexing strategies.
- Consider implementing caching mechanisms for frequently accessed data, especially if the data in the `Currency` table is relatively static.

## Version History

This section is optional but can be included as necessary to track updates and changes to the stored procedure.

---
By following the above specifications, this technical document should provide a comprehensive understanding of the `[dbo].[Currency_Get]` stored procedure and its role in the database system.