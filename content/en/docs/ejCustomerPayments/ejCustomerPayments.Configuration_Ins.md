---
title: Configuration_Ins
weight: 20
---
## Procedure Overview
The stored procedure `[dbo].[Configuration_Ins]` is designed to insert configuration settings into the `[dbo].[Configuration]` table. It allows for the optional encryption of the configuration value before insertion. This procedure addresses the business use case of securely storing sensitive configuration data, such as API keys or connection strings, with an option to encrypt such data for enhanced security.

## Input Parameters
1. **@Key** 
   - **Data Type**: `NVARCHAR(150)`
   - **Purpose**: Represents the configuration item's key or name.
   - **Expected Values/Constraints**: A string up to 150 characters, must be unique within the table to avoid key conflicts.

2. **@Value**
   - **Data Type**: `NVARCHAR(MAX)`
   - **Purpose**: Represents the configuration item's value.
   - **Expected Values/Constraints**: Any string value, which can be a large text for extensive configuration values. If encryption is required, this value will be encrypted before insertion.

3. **@Encrypt**
   - **Data Type**: `BIT`
   - **Purpose**: Indicates whether the configuration value should be encrypted.
   - **Expected Values/Constraints**: `1` for encryption, `0` for no encryption.

4. **@CreateDateTime**
   - **Data Type**: `DATETIME`
   - **Purpose**: The timestamp for when the configuration item is created.
   - **Expected Values/Constraints**: A `DATETIME` value. It should not be in the future.

5. **@CreateUser**
   - **Data Type**: `VARCHAR(50)`
   - **Purpose**: The identifier of the user creating the configuration item.
   - **Expected Values/Constraints**: A string up to 50 characters. If not provided, defaults to the current system user.

## Output Parameters
There are no output parameters for this stored procedure.

## Logic Description
The procedure follows these steps:

1. **Initial Setup**: 
   - Disable rowcount display with `SET NOCOUNT ON`.
   - Check if `@CreateUser` is `NULL`. If it is, set `@CreateUser` to `SYSTEM_USER`, which is the current system user.

2. **Condition Check for Encryption**:
   - If `@Encrypt` is `1` and `@Value` is not `NULL`:
     - Check if the symmetric key `CREDITCARD` is already open in the database `ejCustomerPayments`.
     - If not, open the symmetric key `CREDITCARD` using the certificate `CERT_CREDITCARD`.
     - Encrypt `@Value` using the symmetric key `CREDITCARD` and store it in `@EncryptedValue`.
     - Insert `@Key`, encrypted `@Value` (as `@EncryptedValue`), `@CreateDateTime`, and `@CreateUser` into the `[dbo].[Configuration]` table.
   - Else:
     - Directly insert `@Key`, `@Value`, `@CreateDateTime`, and `@CreateUser` into the `[dbo].[Configuration]` table.

## Dependencies
- **Tables**:
  - `[dbo].[Configuration]`: The target table where configuration data is inserted.
- **Views**:
  - None.
- **Other Stored Procedures**:
  - None.
- **Keys and Certificates**:
  - `CREDITCARD`: A symmetric key used for encryption.
  - `CERT_CREDITCARD`: A certificate used to open the `CREDITCARD` key.

## Example Usage
### Example 1: Inserting a non-encrypted configuration
```sql
EXEC [dbo].[Configuration_Ins] 
    @Key = 'APIEndpoint', 
    @Value = 'https://api.example.com', 
    @Encrypt = 0, 
    @CreateDateTime = GETDATE(),
    @CreateUser = 'admin';
```

### Example 2: Inserting an encrypted configuration
```sql
EXEC [dbo].[Configuration_Ins] 
    @Key = 'APIKey', 
    @Value = '12345ABCDE', 
    @Encrypt = 1, 
    @CreateDateTime = GETDATE();
    -- @CreateUser is not provided and will default to SYSTEM_USER
```

## Error Handling
The procedure does not include explicit error handling such as `TRY-CATCH` blocks. Errors will be managed by the SQL Server’s default error handling mechanism. If additional error handling is desired, `TRY-CATCH` blocks can be introduced to catch and manage any exceptions that arise during execution.

## Performance Considerations
- **Encryption Overhead**: Encrypting values introduces computational overhead. Only use encryption for sensitive data.
- **Indexing**: Ensure the `[Key]` column in the `[dbo].[Configuration]` table is indexed to improve lookup performance.
- **Batch Inserts**: Avoid frequent calls to the procedure with single inserts if possible. Batch processing inserts could reduce transaction overhead and enhance performance.

## Version History
- **Initial Version**: Date Unknown, Created Stored Procedure.

By following this technical document, users can understand the intended use and inner workings of the `[dbo].[Configuration_Ins]` stored procedure, ensuring correct and optimized usage within their SQL Server environment.