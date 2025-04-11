---
title: MessageStore_Exists
weight: 20
---

## Procedure Overview
### Purpose
The `MessageStore_Exists` stored procedure is designed to check the existence of a specific record in the `NotificationMessageStore` table based on the provided payment provider ID, transaction ID, and event type. This procedure is primarily used for validation purposes within the PaymentService system to confirm whether a particular notification message has already been logged in the system.

### Business Use Case
This procedure addresses the need to ensure that duplicate notification messages are not processed. It helps maintain the integrity of notifications by avoiding redundancy and ensuring that each transaction event is uniquely identified and appropriately handled.

## Input Parameters
### Detailed Description of Each Input Parameter

1. **@PaymentProvider (int)**
   - **Data Type**: `int`
   - **Purpose**: This parameter represents the unique identifier for the payment provider.
   - **Expected Values or Constraints**: Must be a valid integer corresponding to an existing payment provider in the system.

2. **@TransactionId (nvarchar(50))**
   - **Data Type**: `nvarchar(50)`
   - **Purpose**: This parameter represents the unique transaction identifier.
   - **Expected Values or Constraints**: Must be a non-null string with a maximum length of 50 characters.

3. **@EventType (varchar(50))**
   - **Data Type**: `varchar(50)`
   - **Purpose**: This parameter represents the event type associated with the transaction.
   - **Expected Values or Constraints**: Must be a non-null string with a maximum length of 50 characters. It should correspond to valid event types predefined in the system.

## Output Parameters
This stored procedure does not have any output parameters. Instead, it returns a scalar result set with a single column.

- **Return Values**:
  - `1` if the specified record exists in the `NotificationMessageStore` table.
  - `0` if the specified record does not exist.

## Logic Description
### Step-by-Step Explanation of the Logic and Flow

1. **Procedure Context Initialization**
   - A variable `@ProcedureContext` of type `VARBINARY(128)` is declared.
   - `@ProcedureContext` is set to a binary representation of the string literal 'PaymentService_Notifications'.
   - The context information is set using `SET CONTEXT_INFO`.

2. **Existence Check**
   - The procedure checks for the existence of a record in the `NotificationMessageStore` table that matches the provided `@PaymentProvider`, `@TransactionId`, and `@EventType`.
   - If such a record exists, the procedure returns `1`.
   - If no such record exists, the procedure returns `0`.

### SQL Flow
```sql
BEGIN
    DECLARE @ProcedureContext VARBINARY(128)
    SET NOCOUNT ON
    SET @ProcedureContext = CAST('PaymentService_Notifications' AS VARBINARY(128))
    SET CONTEXT_INFO  @ProcedureContext

    IF EXISTS (
        SELECT '1'
        FROM NotificationMessageStore
        WHERE PaymentProvider = @PaymentProvider
          AND TransactionId = @TransactionId
          AND EventType = @EventType
    )
        SELECT 1
    ELSE
        SELECT 0
END
```

## Dependencies
### Referenced or Modified Objects

- **Table**: `NotificationMessageStore`
  - **Role**: The table stores notification messages with associated payment provider IDs, transaction IDs, and event types. This procedure queries this table to check for the existence of specified records.

## Example Usage
### Sample SQL Script

```sql
-- Example call to the MessageStore_Exists stored procedure
DECLARE @Result int;

EXEC @Result = [dbo].[MessageStore_Exists]
    @PaymentProvider = 101,
    @TransactionId = 'TXN12345ABC',
    @EventType = 'PAYMENT_SUCCESS';

-- Output the result
SELECT @Result AS RecordExists;
```
### Expected Result
- `1` if a record with the specified parameters exists in the `NotificationMessageStore` table.
- `0` if no such record exists.

## Error Handling
### Explanation of Error Management

- **TRY-CATCH Block**: This stored procedure does not use a `TRY-CATCH` block. Any runtime errors that occur will propagate to the calling application or script.
- **Implicit Error Handling**: As the procedure primarily performs a read operation with straightforward logic, explicit error handling may not be necessary. However, consider adding error handling for more complex scenarios.

## Performance Considerations
### Optimizations and Potential Bottlenecks

- **Indexing**: Ensure that the `NotificationMessageStore` table has appropriate indexing on columns `PaymentProvider`, `TransactionId`, and `EventType` to optimize the search performance.
- **SET NOCOUNT ON**: The use of `SET NOCOUNT ON` helps improve performance by preventing the sending of DONE_IN_PROC messages to the client for each statement in a stored procedure.

## Version History
### Details of Updates or Changes (Optional)
- **Date**: October 10, 2023
  - **Description**: Initial creation of the `MessageStore_Exists` stored procedure.

---

This comprehensive technical document provides a clear and detailed overview of the `MessageStore_Exists` stored procedure, covering all essential aspects from input parameters to performance considerations.