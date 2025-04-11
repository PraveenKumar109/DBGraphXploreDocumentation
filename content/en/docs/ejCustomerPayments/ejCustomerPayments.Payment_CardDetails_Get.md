---
title: Payment_CardDetails_Get
weight: 20
---

## Procedure Overview

**Stored Procedure Name:** `dbo.Payment_CardDetails_Get`

**Purpose:** 
The `Payment_CardDetails_Get` stored procedure retrieves payment card details from the `SecureData` table and `Payment` table based on a given `PaymentId`.

**Functionality:**
This procedure is designed to retrieve and decrypt payment card details. It decrypts the card number if the encryption method is 1 and provides encrypted data otherwise. It also fetches additional information related to the payment such as card expiry date, payment method type, cardholder name, and other details. This procedure is often called by third-party systems, specifically for hotel bookings through Booking.com.

**Business Use Case:**
The procedure is used by the `ThirdParty` component to retrieve card details necessary for processing payments, ensuring secure and efficient access to sensitive payment information.

## Input Parameters

**@PaymentId**

- **Data Type:** `INT`
- **Purpose:** The unique identifier for the payment record.
- **Expected Values or Constraints:** Must be a valid `PaymentId` existing in the `Payment` table.

## Output Parameters

This stored procedure does not have output parameters, but it returns a result set with the following fields:

- **CardNumber (VARCHAR(50)):** Decrypted card number if the encryption method is 1, otherwise protected data.
- **ExpirationDate (DATE):** The expiry date of the payment card.
- **CardType (VARCHAR(50)):** The type of payment method.
- **CardHolderName (VARCHAR(100)):** The name of the cardholder.
- **EncryptionMethod (INT):** The method used for encryption.
- **IsSavedCard (BIT):** Indicates if the card is saved (1 for true, 0 for false).
- **MerchantReference (VARCHAR(50)):** The merchant reference of the payment.
- **PosId (VARCHAR(50)):** The payment source identifier.
- **Channel (VARCHAR(50)):** The payment channel name.

## Logic Description

1. **Set Context Info:** 
    - Sets the context information for the session using `SET CONTEXT_INFO` for tracking and auditing purposes.

2. **Check and Open Symmetric Key:**
    - Checks if the symmetric key `CREDITCARD` is already open in the database `ejCustomerPayments`. 
    - If not, it opens the symmetric key using the certificate `CERT_CREDITCARD`.

3. **Select Query:**
    - Joins the `Payment` and `SecureData` tables on `SecureDataId`.
    - Based on the `@PaymentId`, it retrieves various details including encrypted or decrypted card number, card expiry date, payment method type, cardholder name, encryption method, saved card status, merchant reference, payment source, and channel.
    - If the encryption method is 1, it decrypts the card number using the key and certificate, otherwise, it returns the protected data.

## Dependencies

- **Tables:**
  - `Payment`
    - Holds details of each payment transaction.
  - `SecureData`
    - Stores sensitive encrypted payment data.

- **Certifications and Keys:**
  - `CERT_CREDITCARD`
    - Certificate used to decrypt the card details.
  - `CREDITCARD`
    - Symmetric key used for encryption and decryption.

## Example Usage

```sql
DECLARE @PaymentId INT = 12345;

EXEC dbo.Payment_CardDetails_Get
    @PaymentId = @PaymentId;
```

**Expected Result:**
A result set containing fields:
- CardNumber
- ExpirationDate
- CardType
- CardHolderName
- EncryptionMethod
- IsSavedCard
- MerchantReference
- PosId
- Channel

## Error Handling

The stored procedure does not explicitly use `TRY-CATCH` blocks. However, it assumes the key and certificate are correctly set up. Potential errors that might need to be addressed include:

- Missing or incorrect `PaymentId` leading to no results.
- Errors in decryption if the key or certificate is not correctly configured or opened.

## Performance Considerations

- **Indexing:** Ensure that `PaymentId` is indexed on the `Payment` table for faster retrieval.
- **Key Management:** Frequently accessing and decrypting card data could have performance implications; optimize key management operations.
- **Minimal Result Set:** Only fetch required fields to optimize data transfer and processing.

## Version History

- **18/08/2017 - Initial Creation:** By Customer Payments, purpose to retrieve card details for third-party component.
- **18/08/2017 - Modification:** Added `IsSavedCard` by RogerW due to `PAYNEW-6054`.
- **30/04/2017 - Modification:** Added `MerchantReference` by Satyag due to `PAYNEW-20932`.
- **16/05/2024 - Modification:** Added `PaymentSource` and `Channel` by Deepaky due to `PAYNEW-20998`.

---

This comprehensive document covers all required sections for understanding, utilizing, and maintaining the `Payment_CardDetails_Get` stored procedure. For additional customization or further clarifications, review the underlying database schema and business requirements.