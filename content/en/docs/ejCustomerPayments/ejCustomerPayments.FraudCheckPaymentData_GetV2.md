---
title: FraudCheckPaymentData_GetV2
weight: 20
---
## Procedure Overview

### Purpose
The `FraudCheckPaymentData_GetV2` stored procedure is designed to retrieve detailed payment information required for fraud checks. This procedure consolidates various pieces of data related to a given payment, including payment method details, secure data, and authentication session information. It is used by the Payments Fraud service to append necessary payment data for fraud detection and analysis.

### Functionality
This procedure executes a complex SQL query to extract and format sensitive payment data securely, ensuring that only the necessary details are decrypted and exposed. Its primary function is to facilitate fraud checks by providing comprehensive payment data.

### Business Use Case
The primary use case of this stored procedure is to support fraud detection operations by enriching the fraud analysis process with detailed payment information. It is called frequently and is integral to the Payments Fraud service's operations to prevent fraudulent transactions.

## Input Parameters

| Parameter       | Data Type         | Purpose                                                                                     | Expected Values or Constraints                         |
|-----------------|-------------------|---------------------------------------------------------------------------------------------|-------------------------------------------------------|
| `@PaymentId`    | `bigint`          | Identifies the specific payment record for which details are to be retrieved.              | An existing Payment ID in the database.               |
| `@InteractionType` | `int`         | Specifies the type of interaction for which authentication session data should be retrieved. | A valid Interaction Type ID.                          |
| `@SessionTypes` | `intcollection`   | A read-only table-valued parameter containing session type IDs to filter the session data. | A collection of valid Session Type IDs.               |

## Output Parameters

There are no explicit output parameters for this stored procedure. The result set produced by the SELECT statement serves as the output.

## Logic Description

### Step-by-Step Logic

1. **Initialization and Setup**:
    - **Declare and Set Procedure Context**: 
      - `@ProcedureContext` is declared to store the context information.
      - `SET NOCOUNT ON` prevents the message indicating the number of rows affected by T-SQL statements.
      - `SET CONTEXT_INFO @ProcedureContext` sets the procedure context for identifying the process in logs or debug procedures.

2. **Data Retrieval**:
    - The procedure retrieves various columns from the `Payment`, `SecureData`, `PayerAuthSessionData`, and `PayerAuthSessionType` tables.
    - **Secure Data Handling**:
      - The `SecureData` column is decrypted if the encryption method is 1 using the `DecryptByKeyAutocert` function, otherwise the protected data is used.
    - **Join Conditions**:
      - The `Payment` table is joined with:
        - `SecureData` to get secure payment information.
        - `PayerAuthSessionData` to retrieve authentication session details based on `PaymentId` and `InteractionType`.
        - `PayerAuthSessionType` to get the name of the session type.
        - The table-valued parameter `@SessionTypes` to filter based on session type IDs.
    - **Conditional Data Retrieval**:
      - For `CardCardNumberFirstSixDigits`, the root payment's details are retrieved if the transaction type indicates a refund or credit.
      
3. **Execution**:
    - The result set ordered by `pasd.CreateDateTime DESC` is returned containing the various payment details.

## Dependencies

### Tables
- `dbo.Payment`: Primary table containing payment information.
- `dbo.SecureData`: Table holding secure payment data.
- `dbo.PayerAuthSessionData`: Table containing session data related to authentication.
- `dbo.PayerAuthSessionType`: Table containing types of payer authentication sessions.

### Views
- None.

### Other Stored Procedures
- The procedure directly references table data but does not call other stored procedures.

## Example Usage

```sql
DECLARE @SessionTypes AS intcollection;
INSERT INTO @SessionTypes (Item) VALUES (1), (2), (3);

EXEC [dbo].[FraudCheckPaymentData_GetV2]
    @PaymentId = 123456789,
    @InteractionType = 1,
    @SessionTypes = @SessionTypes;
```

## Error Handling

- **General Error Management**:
  - The procedure does not include explicit error handling mechanisms such as TRY-CATCH blocks.
  - Errors at the SQL engine level (such as invalid input types or missing data) will propagate to the caller.
  
## Performance Considerations

### Optimization Techniques Used
- **Indexed Joins**: Leveraging indexes on the joined columns (`PaymentId`, `SecureDataId`, `InteractionTypeId`, etc.) to improve JOIN performance.
- **Limited Scope of Decryption**: Only decrypting necessary data at the select clause to minimize performance overhead.

### Potential Performance Bottlenecks
- **Large Dataset Joins**: Extensive joins with large tables might cause performance degradation.
- **Decryption Overhead**: Frequent decryption operations can be CPU intensive.

### Recommendations
- **Index Optimization**: Ensure proper indexing on `PaymentId`, `SecureDataId`, and other frequently joined fields.
- **Monitoring and Analysis**: Regularly monitor execution times and analyze query plans for opportunities to improve performance.

## Version History

- **21 Nov 2017**: Initial creation by Roger Woodford.
- **26 June 2018**: Modification by MarcinT (PAYNEW-9217) to add retrieval of `MerchantReference`.
- **25 Feb 2020**: Renaming to V2 and addition of `AuthenticationSessionType` by Ritesh (PAYNEW-10605).

---

This document provides a comprehensive overview of the `FraudCheckPaymentData_GetV2` stored procedure, elucidating its purpose, parameters, logic, usage, dependencies, error handling, and performance considerations.