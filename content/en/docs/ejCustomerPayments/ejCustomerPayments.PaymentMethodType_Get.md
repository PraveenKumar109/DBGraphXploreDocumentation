---
title: PaymentMethodType_Get
weight: 20
---

## Procedure Overview

### Purpose
The `PaymentMethodType_Get` stored procedure retrieves active payment method types from the `PaymentMethodType_View` view. It is designed to extract a list of payment method types that are currently active and available for transaction processing.

### Functionality
- This procedure filters and returns payment method types that are marked as active.
- It sets a contextual information to facilitate auditing and tracing within the `PaymentService_Payment-GetDetails` context.
- The retrieved data includes various attributes pertinent to each payment method type such as its status, requirement flags, and descriptive details.

### Business Use Case
The procedure is typically used in financial and payment systems for obtaining a list of active payment method types that can be presented to users during transactions. It ensures that only active and valid payment methods are processed, thereby maintaining transaction integrity and compliance.

## Input Parameters

This stored procedure does not take any input parameters.

## Output Parameters

This stored procedure does not define any output parameters. However, it returns a result set containing the following columns, all derived from the `PaymentMethodType_View` view:

### Returned Columns
- **Id**: `INT` - Unique identifier for the payment method type.
- **Code**: `VARCHAR` - Code representing the payment method type.
- **Description**: `VARCHAR` - Description of the payment method type.
- **IsCredit**: `BIT` - Flag indicating if the payment method is a credit-based method.
- **IsActive**: `BIT` - Flag indicating if the payment method type is active.
- **PaymentMethodId**: `INT` - Identifier for the payment method.
- **IsIssueNumberRequired**: `BIT` - Indicator if an issue number is required for this payment method.
- **IsStartDateRequired**: `BIT` - Indicator if a start date is required for this payment method.
- **IsSecurityCodeRequired**: `BIT` - Indicator if a security code is required.
- **SecurityCodeLength**: `INT` - Length of the security code if required.
- **IsBankNameRequired**: `BIT` - Indicator if a bank name is required.
- **IsPayerAuthenticationEnabled**: `BIT` - Indicator if payer authentication is enabled.
- **IsPayerAuthenticationMandated**: `BIT` - Indicator if payer authentication is mandated.

## Logic Description

### Step-by-Step Explanation
1. **Context Initialization**: 
   ```sql
   DECLARE @ProcedureContext VARBINARY(128)
   ```
   Initializes a variable to store the procedure's context information.
   
2. **Context Information**: 
   ```sql
   SET @ProcedureContext = CAST('PaymentService_Payment-GetDetails' AS VARBINARY(128))
   SET CONTEXT_INFO  @ProcedureContext
   ```
   - The procedure sets a contextual string 'PaymentService_Payment-GetDetails' which is cast to binary format and then set as the context information. This aids in tracing and auditing by providing context about the current operation.

3. **Selection of Active Payment Method Types**: 
   ```sql
   SET NOCOUNT ON
   SELECT [Id], Code, [Description], IsCredit, IsActive, PaymentMethodId, IsIssueNumberRequired, 
          IsStartDateRequired, IsSecurityCodeRequired, SecurityCodeLength, IsBankNameRequired, 
          IsPayerAuthenticationEnabled, IsPayerAuthenticationMandated
   FROM PaymentMethodType_View
   WHERE PaymentMethodIsActive = 1
     AND IsActive = 1
   ```
   - Uses `SET NOCOUNT ON` to disable the message indicating the number of rows affected.
   - Executes a SELECT statement to retrieve all columns from the `PaymentMethodType_View` where both `PaymentMethodIsActive` and `IsActive` are true.

## Dependencies

### Tables/Views
- **`PaymentMethodType_View`**: The view from which the stored procedure fetches details about payment method types.

No other stored procedures or additional dependencies are referenced or modified.

## Example Usage

### Sample SQL Script

To call the `PaymentMethodType_Get` stored procedure:
```sql
EXEC [dbo].[PaymentMethodType_Get];
```

### Expected Result Set
The result set will include rows of active payment method types, with columns as described above.

## Error Handling

### Error Management
The current version of the stored procedure does not explicitly include error handling mechanisms such as TRY-CATCH blocks. However, the following general considerations are applicable:
- Setting `CONTEXT_INFO` can fail if the context is not properly converted to `VARBINARY`.
- The SELECT statement will fail if `PaymentMethodType_View` does not exist or if there are permission issues.

### Recommendations for Enhanced Error Handling
To improve error handling, consider wrapping the primary logic within a TRY-CATCH block and logging errors as needed:
```sql
BEGIN TRY
    -- Procedure logic here
END TRY
BEGIN CATCH
    -- Error handling logic here
END CATCH
```

## Performance Considerations

### Optimization Techniques
- **Context Setting**: Using `SET CONTEXT_INFO` allows for better auditing without impacting query performance significantly.
- **Filtering on Active Flags**: Ensures that only relevant rows are processed, potentially reducing the amount of data handled.

### Potential Performance Bottlenecks
- Performance may be impacted if `PaymentMethodType_View` is very large and not properly indexed.
- Although `SET NOCOUNT ON` is used to reduce unnecessary network traffic, absence of proper error handling might lead to hidden performance impacts due to silent errors.

### Recommendations for Efficient Use
- Ensure that the `PaymentMethodType_View` is indexed on the `IsActive` and `PaymentMethodIsActive` columns to enhance filtering performance.
- Depending on the frequency of use, consider caching frequently accessed payment method types to reduce repetitive database queries.

## Version History

### Initial Creation
- **Date**: (Enter Creation Date)
- **Description**: Initial version of the `PaymentMethodType_Get` stored procedure.

---

This document provides a comprehensive technical understanding of the `PaymentMethodType_Get` stored procedure, including its purpose, logical flow, dependencies, and considerations for optimized performance and error handling.