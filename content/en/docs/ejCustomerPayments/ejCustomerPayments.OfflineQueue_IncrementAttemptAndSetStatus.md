---
title: OfflineQueue_IncrementAttemptAndSetStatus
weight: 20
---

## Procedure Overview

The `[dbo].[OfflineQueue_IncrementAttemptAndSetStatus]` stored procedure is designed to update records in the `OfflineQueue` table. Specifically, it increments the `NumberOfAttempts` for a given queue entry, updates the entry's status, and sets the update timestamp and user accordingly. This procedure addresses the business use case of tracking and managing processing attempts for items in an offline queue, often used in scenarios where retry mechanisms are pivotal due to intermittent failures in processing.

## Input Parameters

1. **@QueueId** (INT)
   - **Purpose:** The unique identifier for the queue entry to be updated.
   - **Expected Values:** Any valid `INT` value corresponding to an existing `OfflineQueueId` in the `OfflineQueue` table.
   - **Constraints:** Must correspond to an existing record in the `OfflineQueue` table.

2. **@UpdateDateTime** (DATETIME)
   - **Purpose:** The datetime value indicating when the update is made.
   - **Expected Values:** Any valid `DATETIME` value.
   - **Constraints:** Defaults to the current UTC time if not provided.

3. **@UpdateUser** (VARCHAR(50))
   - **Purpose:** The user responsible for making the update.
   - **Expected Values:** Any valid `VARCHAR` value up to 50 characters.
   - **Constraints:** Defaults to the current system user if not provided.

4. **@NewStatus** (INT)
   - **Purpose:** The new status ID to be set for the queue entry.
   - **Expected Values:** Any valid `INT` value corresponding to a valid status in the system.
   - **Constraints:** Should be a valid status identifier in the context of the system's status management.

## Output Parameters

None.

## Logic Description

1. **Variable Declarations:**
   - **@ProcedureContext** (VARBINARY(128)): Used for setting the context information.

2. **Settings:**
   - **SET NOCOUNT ON:** Prevents the message indicating the number of rows affected by a SQL statement from being returned.

3. **Context Setting:**
   - Sets the `@ProcedureContext` to a specific value (`'PaymentService_Payment-Processing-Offline'`), and assigns it to the `CONTEXT_INFO` for tracking purposes.

4. **Default Value Assignments:**
   - If `@UpdateDateTime` is `NULL`, set it to `GETUTCDATE()` (current UTC date and time).
   - If `@UpdateUser` is `NULL`, set it to the current system user (`SYSTEM_USER`).

5. **Update Operation:**
   - Update the specified queue entry in the `OfflineQueue` table, incrementing `NumberOfAttempts`, setting `UpdateDateTime` and `UpdateUser`, and updating the status (`OfflineQueueStatusId`) to `@NewStatus`.

## Dependencies

- **Tables:**
  - **OfflineQueue:** The table being updated by this procedure. This table holds details about offline processing queue entries, including their number of attempts, last update time, the user who made the update, and their current status.

## Example Usage

```sql
-- Example SQL script to call the stored procedure

-- Update the OfflineQueue entry with QueueId 1
DECLARE @QueueId INT = 1;
DECLARE @NewStatus INT = 2;

EXEC [dbo].[OfflineQueue_IncrementAttemptAndSetStatus]
    @QueueId = @QueueId,
    @UpdateDateTime = NULL,         -- Will default to current UTC time
    @UpdateUser = NULL,             -- Will default to system user
    @NewStatus = @NewStatus;        -- New status to be set
```

## Error Handling

- The procedure does not explicitly use TRY-CATCH blocks for error handling. It relies on the SQL Server’s inherent error handling to propagate any exceptions that occur. For improved error management, consider adding comprehensive error handling using TRY-CATCH blocks, custom error messages, and logging mechanisms.

## Performance Considerations

- **SET NOCOUNT ON:** Improves performance by preventing the return of unnecessary row count messages.
- **Default Value Handling:** By reducing the need for checks or external inputs when `NULL` values are provided, the procedure makes efficient use of system resources.
- **Potential Bottlenecks:**
  - **Updates on Large Tables:** Regular updates on large tables might cause performance issues if not indexed properly.
  - **Concurrency:** Should be considered to avoid potential locking problems when multiple processes try to update the same rows concurrently.

## Version History

- **Version 1.0:** Initial version created on [Creation Date]. Added basic functionality for incrementing attempts and setting statuses in the `OfflineQueue`.

This document provides a comprehensive overview of the `[dbo].[OfflineQueue_IncrementAttemptAndSetStatus]` stored procedure, its designed functionality, usage, and technical considerations. For further optimizations and enhanced error handling, future iterations should be considered.
