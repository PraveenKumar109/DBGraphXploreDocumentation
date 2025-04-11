---
title: dbo.bpt_ViewFares
weight: 20
---
## Procedure Overview
The `dbo.bpt_ViewFares` stored procedure is designed to retrieve detailed fare and flight information for a specific flight, identified by its `FlightID`. This procedure is essential for business operations involving fare management, ticket sales analysis, and reporting.

### Functionality
- Retrieve and display flight details such as departure date, airports, and seats sold.
- Fetch fare details including fare class codes, prices, and associated translations.
- Summarize the total seats sold by fare class for the specified flight.

### Business Use Case 
This procedure can be used by airline operational systems to provide:
- Management insights into flight occupancy and fare distribution.
- Sales reports including fare breakdowns and currency conversions.
- Customer service interfaces for responding to fare inquiries.

## Input Parameters
| Parameter  | Data Type | Purpose                                                 | Expected Values / Constraints               |
|------------|-----------|---------------------------------------------------------|---------------------------------------------|
| `@FlightID`| `int`     | The identifier for the flight whose fare details are to be retrieved.| Must be a valid flight ID present in the `ejFlight.dbo.FlightView` table.|

## Output Parameters
This stored procedure does not use explicit output parameters, but it returns three result sets.

## Logic Description
This section provides a step-by-step explanation of the logic and flow within the stored procedure:

1. **Turn off messages count**: 
   ```sql
   set nocount on
   ```

2. **First Result Set**: Select basic flight information filtered by `FlightID`.
   ```sql
   SELECT LocalDepDt = convert(varchar, LocalDepDt, 101), DepAirportCode, ArrAirportCode, FlightNumber, Lid, SeatsSold 
   FROM ejFlight.dbo.FlightView WITH(NOLOCK) 
   WHERE FlightID = @FlightID
   ```

3. **Second Result Set**: Retrieve fare information along with fare class codes and translations.
   ```sql
   SELECT FlightFareID, FlightID, SpecialFareID, ff.FareClassCodeID, FareClassCode, ExtendedPrice, CurrencyCode = 'GBP', AUmin, AU, SeatsSold 
   FROM ejFlight.dbo.FlightFare ff WITH (NOLOCK)
   JOIN dbo.FareClassCode fcc WITH (NOLOCK) 
   ON ff.FareClassCodeID = fcc.FareClassCodeID 
   LEFT OUTER JOIN dbo.FareClasspricetranslation fcpt WITH (NOLOCK)
   ON ff.FareClassCodeID = fcpt.FareClassCodeID AND fcpt.CurrencyCodeID = 4
   WHERE ff.FlightID = @FlightID 
   ORDER BY ff.SpecialFareID DESC, fcpt.ExtendedPrice
   ```

4. **Third Result Set**: Summarize and select the total seats sold by fare class for the specified flight.
   ```sql
   SELECT SeatsSoldByFareClass = SUM(SeatsSold) 
   FROM dbo.FlightFare WITH (NOLOCK) 
   WHERE FlightID = @FlightID
   ```

5. **End the Procedure**: Return a value to indicate successful completion.
   ```sql
   RETURN 0
   ```

## Dependencies
This stored procedure references and queries the following objects:
- **Tables**: 
  - `ejFlight.dbo.FlightView`: Stores information on flight schedules.
  - `ejFlight.dbo.FlightFare`: Contains data on fare components related to flights.
  - `dbo.FareClassCode`: Stores fare class code details.
  - `dbo.FareClasspricetranslation`: Stores translated fare price data for various currencies.

## Example Usage
Here's a sample SQL script for calling the `dbo.bpt_ViewFares` procedure:

```sql
DECLARE @FlightID int = 12345;
EXEC dbo.bpt_ViewFares @FlightID;
```
Expected Results:
1. Basic flight details for FlightID = 12345.
2. Fare details, including fare class codes and GBP prices, for FlightID = 12345.
3. Summarization of total seats sold by fare class for FlightID = 12345.

## Error Handling
- The procedure does not contain explicit error handling mechanisms such as TRY-CATCH blocks.
- To manage errors, consider adding error handling to check for invalid `FlightID` values or for unexpected database errors.

## Performance Considerations
- **NOLOCK** Hints: Used to improve performance by reducing lock contention at the cost of potentially reading uncommitted data.
- **Index Utilization**: Ensure that indexes are properly set on `FlightID` and any other frequently queried fields to optimize performance.
- **Query Optimization**: The procedure sorts and aggregates data, so ensure that the underlying tables are optimized for these operations.

## Version History
(Optional)
- **Version 1.0**: Initial creation on 12/15/04 by Pnuttall.

## Conclusion
The `dbo.bpt_ViewFares` stored procedure is a critical component for retrieving and managing flight fares and details. This document provides a comprehensive overview of its functionality, ensuring effective use and maintenance.