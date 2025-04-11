---
title: MarketAreaCodeAirportCodeAssoc_get
weight: 20
---
## Procedure Overview
The stored procedure `[dbo].[MarketAreaCodeAirportCodeAssoc_get]` is designed to retrieve information associated with market areas and airport codes from the database. This procedure serves the business purpose of querying specific details about the associations between market areas and airport codes, including, optionally, the currency code related to the airport. This is particularly useful for applications needing to display or process these associations in contexts where both market area and airport details are relevant.

## Input Parameters

1. **@MarketAreaCodeID** 
   - **Data Type:** EntityIdentifier
   - **Purpose:** Identifies the specific market area for which the associated airport details are to be queried.
   - **Expected Values/Constraints:** Should be a valid identifier of an existing market area in the database.

2. **@AirportCodeID**
   - **Data Type:** EntityIdentifier
   - **Purpose:** Identifies the specific airport code for the query. This parameter is optional.
   - **Expected Values/Constraints:** Should be a valid identifier of an existing airport code in the database or NULL.

## Output Parameters
There are no explicit output parameters for this stored procedure. Instead, it returns a result set directly containing the queried data.

## Logic Description
The stored procedure follows these steps:

1. **Initialization:** The `SET NOCOUNT ON` directive is used to suppress the number of affected rows message.

2. **Declaration:** An integer variable `@ReturnCode` is declared but not utilized in this version of the procedure.

3. **Conditional Query Execution:** 
   - If `@AirportCodeID` is not NULL:
     - A SELECT statement joins the `MarketAreaCodeAirportCodeAssoc` and `AirportCode` tables to fetch details where both `MarketAreaCodeID` and `AirportCodeID` match the provided values.
   - If `@AirportCodeID` is NULL:
     - A SELECT statement joins the `MarketAreaCodeAirportCodeAssoc`, `AirportCode`, and `CurrencyCode` tables to fetch details based solely on `MarketAreaCodeID`.
     - In addition to the information from the previous query, this also fetches the currency code related data.

4. **Return Code:** The procedure concludes by returning a 0 to indicate successful execution.

## Dependencies
### Tables
1. **dbo.MarketAreaCodeAirportCodeAssoc**
   - Stores the associations between market areas and airport codes.
2. **dbo.AirportCode**
   - Stores information about various airport codes.
3. **dbo.CurrencyCode**
   - Stores currency code details related to airport codes.

### Roles of Tables
- **MarketAreaCodeAirportCodeAssoc:** Provides the linkage between market areas and airport codes.
- **AirportCode:** Supplies additional details (code and name) for the airport codes.
- **CurrencyCode:** Provides the currency information corresponding to airport codes when `@AirportCodeID` is NULL.

## Example Usage
```sql
-- Example 1: Query with both MarketAreaCodeID and AirportCodeID
EXEC [dbo].[MarketAreaCodeAirportCodeAssoc_get] 
    @MarketAreaCodeID = 1, 
    @AirportCodeID = 101;

-- Example 2: Query with only MarketAreaCodeID
EXEC [dbo].[MarketAreaCodeAirportCodeAssoc_get] 
    @MarketAreaCodeID = 1, 
    @AirportCodeID = NULL;
```

## Error Handling
The current stored procedure does not explicitly handle errors (e.g., using TRY-CATCH blocks). If necessary, error handling can be added to manage exceptions such as missing identifiers or other runtime errors during the query execution.

## Performance Considerations
- **NOLOCK Hints:** The use of `WITH (NOLOCK)` hints helps in reducing locking contention by reading uncommitted data, which can improve query performance but may lead to dirty reads.
- **Conditional Logic:** The procedure efficiently separates the logic based on the presence of `@AirportCodeID`, optimizing the query paths accordingly.

## Version History
This section is optional and can be updated with future modifications. As of now, the latest change includes:
- **Date:** Not specified
- **Description:** Added retrieval of `CurrencyCode` when `@AirportCodeID` is NULL as noted by the comment: "--Added BY Akanksha to get currency code as well".

This concludes the detailed technical document for the `[dbo].[MarketAreaCodeAirportCodeAssoc_get]` stored procedure.