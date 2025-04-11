---
title: PaymentTypeCurrency_Extract
weight: 20
---

## Procedure Overview

The stored procedure `dbo.PaymentTypeCurrency_Extract` is designed to extract and return detailed information about payment methods, their corresponding currencies, channels, and payment types associated with a given platform. This stored procedure helps in generating a comprehensive report that integrates payment method details with their corresponding currencies for a specified platform. This is particularly useful for business analysts and financial managers to understand the payment infrastructure across different channels and platforms.

## Input Parameters

### @Platformid
- **Data Type**: `SMALLINT`
- **Purpose**: Identifies the platform for which the payment and currency details need to be extracted.
- **Expected Values or Constraints**: Positive small integers (e.g., `1`, `2`, `3`). The default value is `1`.

## Output Parameters

There are no explicit output parameters. The procedure returns a result set containing the following columns:

- `CurrencyId` (INT): The unique identifier for the currency.
- `CurrencyCode` (VARCHAR(3)): The ISO code of the currency.
- `CurrencySymbol` (NVARCHAR(10)): The symbol representing the currency.
- `CurrencyDescription` (NVARCHAR(50)): Description of the currency.
- `ChannelId` (INT): The unique identifier for the channel.
- `ChannelName` (NVARCHAR(20)): The name of the channel.
- `PaymentMethodId` (INT): The unique identifier for the payment method.
- `PaymentMethodDescription` (NVARCHAR(50)): Description of the payment method.
- `PaymentMethodTypeId` (INT): The unique identifier for the payment method type.
- `PaymentMethodTypeCode` (VARCHAR(20)): Code representing the payment method type.
- `PaymentMethodTypeDescription` (NVARCHAR(50)): Description of the payment method type.

## Logic Description

1. **Set Procedure Context**:
   - Initializes the `@ProcedureContext` variable with a context identifier `'PaymentService_Payment-Downstream'`, then sets the context information for traceability.

2. **Join and Union Operations**:
   - **First UNION SELECT**: Retrieves active payment methods with no associated currency (`CurrencyId IS NULL`) from `PaymentMethodAssoc`, ensuring both payment method associations and currencies are active, and the platform ID matches the input parameter.
   - **Second UNION SELECT**: Retrieves active payment methods with associated currencies (`CurrencyId IS NOT NULL`) from `PaymentMethodAssoc`, also filtered by matching platform ID.
   
3. **Final Data Selection**:
   - Joins the unioned results (`d`) with `Currency`, `Channel`, `PaymentMethod`, and `PaymentMethodType` to enhance the result set with detailed currency, channel, payment method, and payment method type information.
   - Filters records to ensure only active channels, payment methods, and payment method types are included in the result set.

## Dependencies

### Tables
- **PaymentMethodAssoc**: Contains associations between payment methods, types, and channels.
- **Currency**: Holds information about various currencies.
- **Channel**: Stores data about different channels.
- **PaymentMethod**: Contains details about payment methods.
- **PaymentMethodType**: Stores information about types of payment methods.

Each table plays a crucial role in joining and filtering the required data to produce the final output.

## Example Usage

```sql
-- Example of calling the stored procedure
EXEC dbo.PaymentTypeCurrency_Extract @Platformid = 1;
```

The above call will fetch payment method details for platform ID 1.

## Error Handling

There is no explicit error handling (such as `TRY-CATCH` blocks) within this stored procedure. Errors related to invalid platform IDs or inactive references will naturally result in empty result sets without causing procedure failure.

## Performance Considerations

1. **Indexes**: Ensure that indexes on `Platformid`, `IsActive`, `CurrencyId`, and other foreign key columns are properly maintained for optimal query performance.
2. **SET NOCOUNT ON**: Prevents the sending of unnecessary messages to the client, thereby reducing network traffic and improving performance.
3. **Efficient Joins**: The use of inner joins helps in maintaining efficient data retrievals ensuring only matching records are considered.
4. **Data Constraints**: Filtering data with `IsActive=1` ensures that only relevant active data is processed.

## Version History

- **Initial Version**:
  - **Date**: TBD
  - **Description**: Initial creation of the `dbo.PaymentTypeCurrency_Extract` procedure to extract payment method and currency details for a specific platform.

---

This document details the stored procedure `dbo.PaymentTypeCurrency_Extract`, explaining its purpose, input parameters, logic, and usage in depth, along with considerations for performance and error management.
