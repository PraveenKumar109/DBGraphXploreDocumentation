---
title: PaymentDetails_Get
weight: 20
---

## 1. Procedure Overview
The `dbo.PaymentDetails_Get` stored procedure is designed to retrieve payment details from the `Payment` and `PaymentChangeHistory` tables within a specified date range. This procedure joins these tables and returns comprehensive details related to various transactions, facilitating detailed analysis and reporting on payment activities within the given time frame.

### Business Use Case
This stored procedure is particularly useful for financial departments and auditors who need to analyze payment transactions, track statuses, and identify any issues or trends in payment processing over a specific period.

## 2. Input Parameters
The stored procedure accepts two input parameters:

- **@StartDate (DATETIME)**: The start date for filtering the payment records. This parameter defines the beginning of the date range.
  - **Purpose**: To specify the lower limit of the CreationDateTime for the records to be retrieved.
  - **Expected Values**: A valid DATETIME value representing the start date.

- **@EndDate (DATETIME)**: The end date for filtering the payment records. This parameter defines the end of the date range.
  - **Purpose**: To specify the upper limit of the CreationDateTime for the records to be retrieved.
  - **Expected Values**: A valid DATETIME value representing the end date.

## 3. Output Parameters
This stored procedure does not include any output parameters. The results are returned as a result set.

## 4. Logic Description
The following outlines the step-by-step logic implemented within the `dbo.PaymentDetails_Get` stored procedure:

1. **Procedure Context Initialization**:
   - **Declaration**: A variable `@ProcedureContext` of type `VARBINARY(128)` is declared.
   - **Set Context**: The value `PaymentService_Payment-GetDetails` is cast to `VARBINARY(128)` and assigned to `@ProcedureContext`.
   - **Context Info**: The context information is set using `SET CONTEXT_INFO @ProcedureContext`.
   
2. **NOCOUNT**: The `SET NOCOUNT ON` statement is used to prevent the message indicating the number of rows affected by a T-SQL statement from being returned.

3. **Main Query Execution**:
   - **Select Statement**: Various columns are selected from `Payment` (`p`) and `PaymentChangeHistory` (`pch`) tables.
   - **Joins and Filters**: 
     - An inner join is performed between these two tables on the `PaymentId`.
     - The `CreationDateTime` from `PaymentChangeHistory` is filtered between the specified `@StartDate` and `@EndDate` parameters.
   - **NOLOCK Hint**: Both tables are accessed using the `WITH (NOLOCK)` hint to avoid locking and reduce contention.

## 5. Dependencies
The stored procedure references the following tables:

- **dbo.Payment**: Contains detailed information about each payment including transaction types, references, amounts, and PaymentProvider details.
  - **Role**: Provides core information about each payment transaction.
  
- **dbo.PaymentChangeHistory**: Tracks changes in payment statuses and other modifications related to payments.
  - **Role**: Used to filter and retrieve payment records based on the date range provided by the input parameters.

## 6. Example Usage
Here is a sample SQL script demonstrating how to call the `dbo.PaymentDetails_Get` procedure:

```sql
DECLARE @StartDate DATETIME = '2023-01-01 00:00:00'
DECLARE @EndDate DATETIME = '2023-01-31 23:59:59'

EXEC dbo.PaymentDetails_Get
    @StartDate = @StartDate,
    @EndDate = @EndDate
```

### Expected Result
This call will return a result set containing all payment records from `dbo.Payment` and `dbo.PaymentChangeHistory` tables where the `CreationDateTime` falls between `'2023-01-01 00:00:00'` and `'2023-01-31 23:59:59'`.

## 7. Error Handling
The stored procedure does not explicitly implement error handling mechanisms such as `TRY-CATCH` blocks. It is recommended to handle errors at the calling point or expand the procedure to include error handling logic.

## 8. Performance Considerations
- **NOLOCK Hint**: The use of `WITH (NOLOCK)` can improve performance by reducing locking contention but comes at the risk of reading uncommitted data (dirty reads).
- **Indexes**: Ensure that appropriate indexes exist on `PaymentId` and `CreationDateTime` columns to optimize join and filter operations.
- **Context Info**: Setting context information can be useful for tracking and debugging but adds an additional step that may slightly impact performance.

## 9. Version History (Optional)
- **Version 1.0**: Initial version created on [Insert Date], includes the basic implementation for fetching payment details within a specified date range.

---

This document provides a comprehensive overview of the `dbo.PaymentDetails_Get` stored procedure, facilitating its understanding and proper utilization within the system.