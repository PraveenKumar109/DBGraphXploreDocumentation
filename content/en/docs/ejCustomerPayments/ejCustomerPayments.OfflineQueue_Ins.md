---
title: OfflineQueue_Ins
weight: 20
---

## Procedure Overview

### Purpose
The stored procedure [dbo].[OfflineQueue_Ins] is designed to insert a new record into the `OfflineQueue` table. Its primary function is to queue payment transactions that require offline processing. This is useful for handling situations where online processing is not available or fails, ensuring that the transactions can be processed later.

### Business Use Case
The stored procedure is valuable in financial systems where maintaining the reliability and continuity of transaction processing is critical, even when the system experiences downtime or connectivity issues. The use of this procedure ensures that transactions are not lost and can be processed when the system recovers, thereby maintaining data integrity and transaction consistency.

## Input Parameters

| Parameter Name           | Data Type    | Purpose                                                                                     | Expected Values or Constraints                                                                                                    |
|--------------------------|--------------|---------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| @PaymentId               | INT          | Unique identifier for the payment transaction.                                              | Must be a valid ID from the associated payment system.                                                                             |
| @TransactionType         | INT          | Type of transaction being processed.                                                        | Must correspond to predefined transaction types in the system.                                                                     |
| @OfflineQueueStatusId    | INT          | Status identifier for the queue entry, indicating current processing state.                  | Should match valid status IDs defined in the system's status table.                                                                |
| @NumberOfAttempts        | INT          | Number of attempts made to process the transaction.                                          | Must be a non-negative integer, typically starting at 0.                                                                           |
| @CreateDateTime          | DATETIME     | Timestamp for when the queue entry is created.                                               | If NULL, defaults to the current UTC date and time (GETUTCDATE()).                                                                  |
| @CreateUser              | VARCHAR(50)  | Username of the person or system creating the entry.                                         | If NULL, defaults to the current user executing the procedure (SYSTEM_USER).                                                        |

## Output Parameters

This procedure does not have any output parameters.

## Logic Description

1. **Context Information**: 
   - Initialize `@ProcedureContext` with a value representing the procedure context.
   - Set the execution context using `SET CONTEXT_INFO` to track the source of execution.

2. **Default Value Assignments**:
   - If `@CreateDateTime` is NULL, set it to the current UTC date and time using `GETUTCDATE()`.
   - If `@CreateUser` is NULL, set it to the current system user using `SYSTEM_USER`.

3. **Insert Operation**:
   - Insert a new record into the `OfflineQueue` table with the provided parameter values. Ensures that `PaymentId`, `TransactionType`, `OfflineQueueStatusId`, `NumberOfAttempts`, `CreateDateTime`, and `CreateUser` are accurately recorded in the table.

```sql
INSERT INTO [OfflineQueue]
           ([PaymentId],
            [TransactionType],
            [OfflineQueueStatusId],
            [NumberOfAttempts],
            [CreateDateTime],
            [CreateUser])
VALUES     (@PaymentId,
            @TransactionType,
            @OfflineQueueStatusId,
            @NumberOfAttempts,
            @CreateDateTime,
            @CreateUser)
```

## Dependencies

### Tables
- **OfflineQueue**: The main table where the offline queue entries are stored.

### Views
- None

### Other Stored Procedures
- None

## Example Usage

```sql
-- Example usage of the stored procedure [dbo].[OfflineQueue_Ins]

DECLARE @PaymentId INT = 12345
DECLARE @TransactionType INT = 1
DECLARE @OfflineQueueStatusId INT = 0
DECLARE @NumberOfAttempts INT = 0

EXEC [dbo].[OfflineQueue_Ins]
    @PaymentId = @PaymentId,
    @TransactionType = @TransactionType,
    @OfflineQueueStatusId = @OfflineQueueStatusId,
    @NumberOfAttempts = @NumberOfAttempts,
    @CreateDateTime = NULL, -- Optional; defaults to current UTC time
    @CreateUser = NULL      -- Optional; defaults to system user
```

## Error Handling

The procedure currently does not include explicit error handling mechanisms like TRY-CATCH blocks. Errors such as data type mismatches or constraint violations will be handled by SQL Server's default error handling. To improve error management, consider incorporating TRY-CATCH blocks to capture and handle exceptions within the procedure.

## Performance Considerations

### Optimization Techniques
- **SET NOCOUNT ON**: Used to prevent the sending of DONE_IN_PROC messages to the client for each statement, which can reduce network traffic and improve performance.

### Potential Performance Bottlenecks
- **Transaction Volume**: High volumes of concurrent inserts into the `OfflineQueue` table may lead to contention and locking issues.
- **Indexing**: Ensure appropriate indexing on `OfflineQueue` for quicker retrieval and insertion.

### Recommendations
- Regularly monitor performance and analyze execution plans.
- Ensure appropriate indexing on fields used frequently in WHERE clauses or joins.
- Implementing error handling to manage exceptions and log useful diagnostic information.

## Version History

### Initial Version
- **Date**: 2023-10-01
- **Description**: Initial creation of the stored procedure [dbo].[OfflineQueue_Ins].

This document provides a comprehensive overview of the [dbo].[OfflineQueue_Ins] stored procedure, detailing its functionality, usage, and best practices for maintenance and performance.