---
title: AdyenCardBrand_Get
weight: 20
---
### 1. Procedure Overview
**Purpose:**  
The stored procedure `[dbo].[AdyenCardBrand_Get]` is designed to retrieve a list of card types and their corresponding card brands from the `AdyenCardBrand` table. This procedure is utilized primarily within payment processing systems to identify and categorize different card brands based on the card type.

**Business Use Case:**  
This stored procedure is essential for payment service applications that need to populate dropdowns or validate card details in real-time during the payment processing workflow. It supports identifying various card brands such as VISA, MasterCard, etc., by their card type codes.

### 2. Input Parameters
This stored procedure does not accept any input parameters.

### 3. Output Parameters
This stored procedure does not define any output parameters explicitly. However, it returns a result set with the following columns:
- `CardType` (Data Type: `VARCHAR`): The type of card (e.g., credit, debit).
- `CardBrand` (Data Type: `VARCHAR`): The brand of the card (e.g., VISA, MasterCard).

### 4. Logic Description
The stored procedure `[dbo].[AdyenCardBrand_Get]` performs the following logical steps:

1. **Declare Variable `@ProcedureContext`:**  
   A variable `@ProcedureContext` of type `VARBINARY(128)` is declared. This variable is used to store context information for tracking and auditing purposes.
   
2. **Set NOCOUNT ON:**  
   Disables the message that typically displays the number of rows affected by a T-SQL statement, which improves performance by reducing network traffic.

3. **Assign Context Information:**  
   Sets `@ProcedureContext` to the binary representation of the string 'PaymentService_Payment-Processing', which is subsequently set as the context info.
   ```sql
   SET @ProcedureContext = CAST('PaymentService_Payment-Processing' AS VARBINARY(128))
   SET CONTEXT_INFO @ProcedureContext
   ```
   
4. **Select Statement:**   
   Executes a `SELECT` query to retrieve the `CardType` and `CardBrand` columns from the `AdyenCardBrand` table.
   ```sql
   SELECT CardType, CardBrand FROM AdyenCardBrand
   ```

### 5. Dependencies
**Tables:**
- `AdyenCardBrand`: This table contains mappings of card types to their respective card brands. The stored procedure reads from this table to provide the required information.

**Procedures or Views:**
- None directly referenced.

### 6. Example Usage
To call the stored procedure and retrieve the results, you can use the following SQL script:

```sql
EXEC [dbo].[AdyenCardBrand_Get]
```

**Expected Result:**  
This will return a result set consisting of two columns: `CardType` and `CardBrand`, populated with the corresponding data from the `AdyenCardBrand` table.

### 7. Error Handling
This stored procedure does not implement explicit error handling mechanisms like TRY-CATCH blocks. Basic safeguards like setting context information for tracking are present, but no specific error messages or codes are managed within the procedure. Any errors that occur will be SQL runtime errors.

### 8. Performance Considerations
- **SET NOCOUNT ON:** Ensures that the procedure does not return the number of rows affected message, which saves bandwidth.
- **Context Information:** While setting context info is useful for auditing, its performance impact is minimal but should be consciously used.
- **Select Query:** The `SELECT` statement is straightforward and is expected to perform efficiently assuming appropriate indexing on the `AdyenCardBrand` table.

### 9. Version History
**Initial Version:**
- **Date:** [Insert Date]
- **Description:** Initial creation of the stored procedure to fetch card types and brands from the `AdyenCardBrand` table.

---

This comprehensive technical document serves to explain the functionality, usage, and considerations for the `[dbo].[AdyenCardBrand_Get]` stored procedure, aiding developers and database administrators in understanding and utilizing the procedure effectively.