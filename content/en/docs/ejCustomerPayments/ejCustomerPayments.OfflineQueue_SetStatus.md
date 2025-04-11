---
title: OfflineQueue_SetStatus
weight: 20
---


## Procedure Overview
The `[dbo].[OfflineQueue_SetStatus]` stored procedure is designed to update the status of a payment record in the `OfflineQueue` table. This is typically used in the context of processing offline payments. The procedure checks if the current status of the payment matches an expected status before updating it to a new status. 

## Input Parameters
### `@PaymentId`
- **Data Type**: `INT`
- **Purpose**: The unique identifier of the payment record to be updated.
- **Expected Values or Constraints**: Should be a valid `PaymentId` present in the `OfflineQueue` table.

### `@ExpectedStatus`
- **Data Type**: `INT`
- **Purpose**: The status ID that the payment record must currently have for the update to proceed.
- **Expected Values or Constraints**: Should match a valid status ID in the `OfflineQueueStatusId` column.

### `@NewStatus`
- **Data Type**: `INT`
- **Purpose**: The new status ID to set for the payment record.
- **Expected Values or Constraints**: Should be a valid status ID.

### `@UpdatedCount`
- **Data Type**: `INT`
- **Purpose**: Output parameter that will hold the count of rows affected by the update operation.
- **Expected Values or Constraints**: None.

### `@UpdateDateTime`
- **Data Type**: `DATETIME`
- **Purpose**: The date and time when the status was updated. Defaults to the current UTC date and time if not provided.
- **Expected Values or Constraints**: Valid date and time value.

### `@UpdateUser`
- **Data Type**: `VARCHAR(50)`
- **Purpose**: The username of the person or system making the update. Defaults to the system user if not provided.
- **Expected Values or Constraints**: Should be a string with a maximum length of 50 characters.

## Output Parameters
### `@UpdatedCount`
- **Data Type**: `INT`
- **Significance**: Provides the number of rows that were affected by the update. Useful for confirming whether the intended update was successful.

## Logic Description
1. **Context Information Setup**: 
   - Declares and sets the `@ProcedureContext` with a specific value indicating the context of the procedure.
   - This is useful for tracking and diagnostic purposes.

2. **Default Value Assignments**: 
   - If `@UpdateDateTime` is not provided, it defaults to the current UTC date and time (`GETUTCDATE()`).
   - If `@UpdateUser` is not provided, it defaults to the current system user (`SYSTEM_USER`).

3. **Update Operation**: 
   - The `UPDATE` statement targets the `OfflineQueue` table.
   - It sets the `OfflineQueueStatusId`, `UpdateDateTime`, and `UpdateUser` for the record matching the provided `@PaymentId` and `@ExpectedStatus`.

4. **Affected Rows Count**: 
   - The `@@rowcount` is retrieved to determine the number of rows affected by the `UPDATE` operation.
   - This count is passed back using the `@UpdatedCount` output parameter.

5. **Return Statement**: 
   - Procedure ends with a `RETURN 0` to indicate successful completion.

## Dependencies
- **Tables**: 
  - `OfflineQueue`: The table that holds the payment records and their statuses.

## Example Usage
```sql
DECLARE @RowsAffected INT;

EXEC [dbo].[OfflineQueue_SetStatus]
    @PaymentId = 1234,
    @ExpectedStatus = 1,
    @NewStatus = 2,
    @UpdatedCount = @RowsAffected OUTPUT;

SELECT @RowsAffected AS RowsAffected;
```
In this example, the call updates the payment record with `PaymentId = 1234` from status `1` to status `2` and returns the number of rows affected.

## Error Handling
- This procedure does not explicitly use `TRY-CATCH` blocks or raise errors. Any runtime errors (e.g., from the `UPDATE` statement) will propagate to the caller.
- It's recommended to handle potential errors in the application code calling this procedure.

## Performance Considerations
- **Indexes**: Ensure appropriate indexing on the `PaymentId` and `OfflineQueueStatusId` columns to optimize the `UPDATE` operation.
- **Resource Usage**: The procedure is efficient for its purpose but could be further optimized depending on the size of the `OfflineQueue` table and frequency of updates.

## Version History
- **Initial Creation**: [Date not provided] - Created the initial version of the procedure.
  
This concludes the technical documentation for the `[dbo].[OfflineQueue_SetStatus]` stored procedure. Any updates or modifications should also be documented in the Version History section.