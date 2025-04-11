---
title: CreditAccount_Authorize
weight: 20
---

## Procedure Overview
**Purpose:**
The stored procedure `CreditAccount_Authorize` is responsible for authorizing credit transactions based on reservation and credit account information. It handles the validation and insertion of credit authorization data into the system. The business use case involves ensuring that credit, refund, and budget accounts are managed accurately and securely during reservation transactions, supporting both credit account verification and creation.

**Functionality:**
- Verifies the existence of credit accounts and their validity.
- Checks passwords for existing accounts.
- Authenticates and authorizes payment amounts against credit or refund accounts.
- Creates new credit accounts if they do not exist and if certain conditions are met.
- Manages error conditions related to account existence, password mismatch, and others.
- Converts currency and calculates budget and remaining amounts.
  
## Input Parameters
| Parameter Name              | Data Type           | Purpose                                                                   | Expected Values / Constraints                             |
|-----------------------------|---------------------|---------------------------------------------------------------------------|----------------------------------------------------------|
| `@ReservationID`            | `int`               | The unique identifier for the reservation.                                | Positive integer                                          |
| `@CreditCardNumber`         | `varchar(50)`       | The credit card number associated with the account.                       | Valid credit card number or `NULL`                        |
| `@ExpirationDt`             | `TransactionDateTime` | The expiration date of the transaction.                                 | Valid date-time value or `NULL`                            |
| `@PaymentAmount`            | `money`             | The amount to be credited or refunded.                                    | Positive or negative monetary value                       |
| `@Password`                 | `varchar(20)`       | The password for the credit account.                                      | Valid password string                                     |
| `@DisplayCurrencyCodeID`    | `EntityIdentifier`  | Currency code identifier for display purposes.                            | Valid currency code identifier                            |
| `@ExchangeRateGroupID`      | `EntityIdentifier`  | Identifier for the exchange rate group.                                   | Valid exchange rate group identifier                      |
| `@CreditFileTypeCodeID`     | `EntityIdentifier`  | Identifier for the type of credit file (e.g., credit, refund).            | `1`, `2`, `3`, `4`                                         |
| `@AgentID`                  | `EntityIdentifier`  | Identifier for the agent processing the transaction.                      | Valid agent identifier                                    |
| `@MemberID`                 | `EntityIdentifier`  | Member identifier associated with the account.                            | Valid member identifier or `NULL`                        |
| `@ReasonCode`               | `char(3)`           | Code specifying the reason for the transaction.                           | Valid 3-character reason code or `NULL`                   |
| `@ReasonComment`            | `char(50)`          | Additional comments regarding the reason for the transaction.             | Valid comment string or `NULL`                            |

## Output Parameters
This procedure does not have explicit output parameters, but it returns data in XML format based on successful or failed operations.

## Logic Description
**Main Steps:**
1. Set initial variables and perform setup operations.
2. Verify if the credit or refund file account exists based on `@CreditFileTypeCodeID`.
3. Check the password (if applicable) and validate the account.
4. If account validation fails, raise an error and terminate with an appropriate error code.
5. If the credit account exists and has an outstanding balance (negative), insert details into the `CreditAccountDetail` table.
6. If the credit account does not exist and has an outstanding balance (negative), create a new credit account.
7. If the account exists and meets certain conditions, call a stored procedure to calculate authorized amount (`CreditAccount_CalculateAuthorized`).

**Detailed Operations:**
- The stored procedure uses several `SELECT`, `IF`, and `EXECUTE` statements to check conditions, perform currency conversions, insert data into tables, and handle errors.
- Password validation involves checking for both `NULL` and non-`NULL` passwords in `CreditAccount` table.
- Currency conversion and budget calculations are performed with custom user-defined functions (`udf_ConvertMoney`) from other database modules.
- The procedure uses XML format to return results of operations, including authorized amounts, account details, and balance information.

## Dependencies
- **Tables Referenced:**
  - `dbo.CreditAccount`
  - `dbo.CreditAccountDetail`
  - `ejReservation.dbo.CreditAccount`
  - `ejReservation.dbo.CreditAccountDetail`
  - `ejReservation.dbo.CreditFileTypeCode`

- **Stored Procedures/Critical Functions Called:**
  - `ejKeyServer.dbo.PrimaryKeyNextValue_GetNextValue`
  - `ejReservation.dbo.CreditAccountDetail_ins`
  - `ejReservation.dbo.CreditAccount_ins`
  - `ejReservation.dbo.CreditAccount_CalculateAuthorized`
  - `dbo.udf_ConvertMoney` (function for currency conversion)

## Example Usage
```
-- Example call to the CreditAccount_Authorize stored procedure
EXEC dbo.CreditAccount_Authorize
    @ReservationID = 12345,
    @CreditCardNumber = '4111111111111111',
    @ExpirationDt = '2023-12-31 23:59:59',
    @PaymentAmount = -200.00,
    @Password = 'password123',
    @DisplayCurrencyCodeID = 840,
    @ExchangeRateGroupID = 1,
    @CreditFileTypeCodeID = 3,
    @AgentID = 110,
    @MemberID = 99999,
    @ReasonCode = 'CXL',
    @ReasonComment = 'Cancelled reservation'
```

## Error Handling
- The procedure uses `RAISERROR` to raise specific error messages and return error codes when validation fails:
  - If the credit file account does not exist, it returns error code `60203` or `60204` based on `@CreditFileTypeCodeID`.
  - If no budget item is found for the specified account number/password, it returns error code `60201`.
- Error handling ensures that invalid or unauthorized transactions are caught and reported early in the process.

## Performance Considerations
- **Optimization Techniques:**
  - Queries utilize `WITH (NOLOCK)` hints to prevent locking issues and improve performance during reads.
  - Critical operations are grouped and minimized to reduce the number of database reads/writes.
  - Currency conversion and calculation functions are utilized to minimize procedural logic load.
- **Potential Bottlenecks:**
  - Large tables or frequent access to `CreditAccount` and `CreditAccountDetail` might result in slower performance; ensure proper indexing.
  - Complex join operations and nested procedures might increase execution time; consider reviewing execution plans and refining queries.
- **Recommendations:**
  - Regularly monitor and update statistics for the dependent tables.
  - Consider indexing frequently queried columns to improve lookup speeds.
  - Review and optimize user-defined functions (e.g., `udf_ConvertMoney`) for performance enhancements.

## Version History (Optional)
- **Initial Version:**
  - Date: 2023-10-01
  - Description: Initial release of the `CreditAccount_Authorize` stored procedure to handle authorization and credit account management for reservations.

---

This comprehensive technical document outlines the purpose, structure, and detailed operations of the `CreditAccount_Authorize` stored procedure, facilitating understanding and effective utilization by DBAs and developers.