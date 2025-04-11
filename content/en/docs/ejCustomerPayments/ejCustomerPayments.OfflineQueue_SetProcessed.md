---
title: OfflineQueue_SetProcessed
weight: 20
---

### Procedure Overview
The `OfflineQueue_SetProcessed` stored procedure is designed to update the status of an entry in the `OfflineQueue` table. This procedure is particularly used in scenarios where an offline processing queue needs to mark an entry as processed. The procedure updates the status, processing end time, last update time, and the user who performed the update for a specified queue entry.

### Input Parameters
- **@QueueId** (INT): 
  - **Purpose**: Specifies the unique identifier of the queue entry to be updated.
  - **Expected Values**: Must be a valid entry ID from the `OfflineQueue` table.
  - **Constraints**: Cannot be NULL.

- **@OfflineQueueStatusId** (INT):
  - **Purpose**: Sets the new status for the queue entry.
  - **Expected Values**: Must be a valid status ID corresponding to the status in the business context.
  - **Constraints**: Cannot be NULL.

- **@UpdateDateTime** (DATETIME, Default = NULL):
  - **Purpose**: Specifies the date and time of the update. If not provided, the current UTC date and time is used.
  - **Expected Values**: A valid DATETIME value.
  - **Constraints**: Nullable, defaults to GETUTCDATE() if not provided.

- **@UpdateUser** (VARCHAR(50), Default = NULL):
  - **Purpose**: Specifies the user making the update. If not provided, the system user is used.
  - **Expected Values**: A valid username up to 50 characters.
  - **Constraints**: Nullable, defaults to SYSTEM_USER if not provided.

### Output Parameters
None.

### Logic Description
1. **Variable Initialization**:
   - A variable `@ProcedureContext` is declared and set to store contextual information for debugging or logging purposes.
   
2. **Context Information Setup**:
   - The procedure sets `NOCOUNT ON` to prevent the message indicating the number of rows affected by SQL statements.
   - `@ProcedureContext` is set using the `CAST` function with relevant context information.
   - `CONTEXT_INFO` is set to `@ProcedureContext` for tracing purposes.
   
3. **Default Parameter Handling**:
   - If `@UpdateDateTime` is NULL, it is set to the current UTC date and time using `GETUTCDATE()`.
   - If `@UpdateUser` is NULL, it is set to the current system user using `SYSTEM_USER`.

4. **Updating the OfflineQueue Entry**:
   - The `OfflineQueue` table is updated where `OfflineQueueId` matches the provided `@QueueId`.
   - The columns `OfflineQueueStatusId`, `ProcessingEnded`, `UpdateDateTime`, and `UpdateUser` are updated with the provided or default values.

### Dependencies
- **Tables**:
  - `OfflineQueue`: This table is updated by the stored procedure to set the processing status and other related information.

### Example Usage
```sql
-- Example call to OfflineQueue_SetProcessed
EXEC dbo.OfflineQueue_SetProcessed
    @QueueId = 123,
    @OfflineQueueStatusId = 2,
    @UpdateDateTime = '2023-10-01 10:00:00',
    @UpdateUser = 'admin_user';

-- Example call with default values for optional parameters
EXEC dbo.OfflineQueue_SetProcessed
    @QueueId = 124,
    @OfflineQueueStatusId = 3;
```

### Error Handling
The procedure does not implement explicit error handling mechanisms such as TRY-CATCH blocks. Errors will be handled implicitly by SQL Server, which will raise an error if an issue occurs (e.g., invalid `@QueueId`, constraints violations). For more robust error handling, consider adding TRY-CATCH blocks and custom error messages.

### Performance Considerations
- **Optimization Techniques**: The procedure sets `NOCOUNT ON`, which can improve performance by reducing overhead from extra messages.
- **Potential Bottlenecks**: The performance can be affected by the number of updates required and the size of the `OfflineQueue` table.
- **Recommendations**: Ensure that `OfflineQueueId` has an index to improve lookup performance.

### Version History
- **Initial Version**: 
  - **Date**: YYYY-MM-DD
  - **Description**: Initial creation and implementation of the stored procedure.

This document provides a thorough outline and understanding of the `OfflineQueue_SetProcessed` stored procedure, ensuring clarity for developers, DBAs, and other stakeholders involved in maintaining or utilizing this SQL resource.