---
title: PaymentMethodAssoc_GetV2
weight: 20
---

## Procedure Overview

**Purpose:**
The stored procedure `[dbo].[PaymentMethodAssoc_GetV2]` retrieves active payment methods along with their associated details from the system. This includes information such as payment method descriptions, types, currency information, channel information, and various flags related to the payment method's behavior and requirements.

**Functionality:**
The procedure extracts relevant payment methods from the `PaymentMethodAssoc_View` view, ensuring that only active and valid entries are returned. This procedure is typically used by payment processing modules or financial reporting tools to fetch the current set of available payment methods that can be offered to users or integrated into transactions.

**Business Use Case:**
The main use case of this procedure is to provide a reliable source of valid and active payment methods for various internal and external financial operations. It ensures that the list of payment methods aligns with current business rules and regulatory requirements.

## Input Parameters

This stored procedure does not rely on any input parameters.

## Output Parameters

This stored procedure does not have any output parameters.

## Logic Description

1. **Initialization:**
   - Declares a variable `@ProcedureContext` to store the context information for the procedure.
   - Sets `NOCOUNT` on to avoid sending extra result sets to the client.

2. **Set Context:**
   - Sets the `CONTEXT_INFO` to label the operation with a specific context tag (`'PaymentService_PaymentConfiguration'`).

3. **Select Query:**
   - The procedure selects various columns from the `PaymentMethodAssoc_View` view.
   - It includes several payment method-related columns, such as `PaymentMethodId`, `PaymentMethodDescription`, `CurrencyCode`, `ChannelName`, etc.

4. **Filtering Conditions:**
   - Filters the records to ensure only active payment methods are returned:
     - `PaymentMethodIsActive = 1`
     - `PaymentMethodTypeIsActive = 1`
     - `ISNULL(CurrencyIsActive, 1) = 1`
     - `ISNULL(ChannelIsActive, 1) = 1`
     - `ISNULL(PlatformIsActive, 1) = 1`

## Dependencies

**Tables and Views:**
- `PaymentMethodAssoc_View`: This view provides the consolidated data about payment methods. The filtering logic is applied here to retrieve active payment methods.

## Example Usage

```sql
-- Example SQL script to call the stored procedure
EXEC [dbo].[PaymentMethodAssoc_GetV2];
```

### Expected Results
The procedure will return a result set with the following columns (examples of the expected data are shown for a better understanding):

| PaymentMethodId | PlatformId | PaymentMethodDescription | PaymentMethodTypeId | PaymentMethodTypeDescription | PaymentMethodTypeCode | PaymentMethodIsCanBeSaved | CurrencyId | CurrencyCode | Channel | ChannelName | PaymentMethodTypeIsCredit | IsAvailableOffline | IsActive | IsIssueNumberRequired | IsStartDateRequired | IsSecurityCodeRequired | SecurityCodeLength | IsBankNameRequired | IsPayerAuthenticationEnabled | IsPayerAuthenticationMandated | CountryCode | IsIbanRequired |
|-----------------|------------|---------------------------|---------------------|-----------------------------|-----------------------|---------------------------|------------|--------------|---------|--------------|----------------------------|-------------------|----------|-----------------------|---------------------|------------------------|-------------------|-------------------|----------------------------|----------------------------|-------------|----------------|
| 1               | 10         | Credit Card               | 20                  | Card                        | CC                    | 1                         | 5          | USD          | 3       | Online       | 1                          | 0                 | 1        | 0                     | 0                   | 1                      | 3                 | 0                 | 1                          | 1                          | US          | 0              |

## Error Handling

This stored procedure does not include explicit error handling using `TRY-CATCH` blocks. It leverages SQL Server's default error handling mechanism. Any runtime errors encountered during its execution will cause the procedure to fail, and an error message will be returned to the calling application.

## Performance Considerations

- **Indexes:** Ensure that the underlying view `PaymentMethodAssoc_View` is properly indexed, especially on columns used in the `WHERE` clause (e.g., `PaymentMethodIsActive`, `PaymentMethodTypeIsActive`, etc.) to optimize query performance.
- **NOCOUNT:** The `SET NOCOUNT ON` statement avoids sending unnecessary count message to the client, which can slightly improve performance by reducing network traffic.
- **CONTEXT_INFO:** The usage of `CONTEXT_INFO` helps in tracking and auditing but has negligible performance impact.

## Version History

- **12 Jul 2017:** Created by `<Author Name>`.
- **18 Jul 2019:** Modified by `Marcint` (PAYNEW-12904) - Added parameter `@PlatformId` with a default value of 1 (eRes).
- **23 Oct 2019:** Modified by `Marcint` (PAYNEW-13945) - Removed parameter `@PlatformId`, removed filtering on `PlatformId`, and retrieving column `PlatformId`.
- **21 Oct 2020:** Modified by `HebertGo` (PAYNEW-15950) - Updated the response to return correct information in line with SEPA DD.

This detailed technical document provides a comprehensive view of the stored procedure `[dbo].[PaymentMethodAssoc_GetV2]`, covering its purpose, functionality, and critical aspects for proper understanding and maintenance.