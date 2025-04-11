---
title: Flight_SetInfantSeatsSold
weight: 20
---
## Procedure Overview
The stored procedure `dbo.Flight_SetInfantSeatsSold` is designed to manage and update the infant traveler inventory for a specific flight. It increases the number of infant travelers sold when a new booking is made. This procedure ensures that the updated count does not exceed the available capacity for infants on the flight. 
The main business use case includes managing infant bookings, particularly in scenarios such as free transfers, ghost flight consolidation, and modifications through various booking portals.

## Input Parameters
- **@FlightID** (ENTITYIDENTIFIER): 
  - **Purpose**: Identifies the specific flight for which the infant traveler count needs to be updated.
  - **Expected Values**: A valid flight identifier present in the system.
  
- **@NumInfantTravelers** (INT): 
  - **Purpose**: Specifies the number of additional infant travelers to be added to the current infant traveler count.
  - **Expected Values**: A positive integer greater than 0. Should not be NULL.

## Output Parameters
None.

## Logic Description
1. **Early Exit Validations**:
   - The procedure starts by checking if `@NumInfantTravelers` is NULL or less than or equal to 0. If the condition is met, an error is raised and the procedure exits.
   
2. **Availability Check**:
   - The procedure calculates the available infant seats by querying the `flightcap` table. If there are no available infant seats (i.e., `@InfAvailable` <= 0), an error is raised and the procedure exits.
   
3. **Updating Infant Traveler Count**:
   - The procedure updates the `infSold` column in the `flightcap` table by adding `@NumInfantTravelers` to the current count for the specified `@FlightID`.
   - It then checks if the update operation affected any rows. If no rows were updated, an error is raised indicating that the update failed.

4. **Error Handling**:
   - Errors are managed using `RAISERROR` with specific messages indicating the type of issue encountered (e.g., invalid input, no available infant seats, unsuccessful update).

## Dependencies
- **Tables**:
  - `ejflight.dbo.flightcap`: This table stores the flight details, including the infant capacity (`InfCount`) and the number of sold infant seats (`InfSold`).

## Example Usage
```sql
-- Example usage of Flight_SetInfantSeatsSold procedure
DECLARE @FlightID INT = 12345;  -- Example FlightID
DECLARE @NumInfantTravelers INT = 2;  -- Example number of infant travelers to add

-- Call the stored procedure
EXEC dbo.Flight_SetInfantSeatsSold @FlightID, @NumInfantTravelers;
```

## Error Handling
The procedure uses `RAISERROR` to manage errors:
- If `@NumInfantTravelers` is NULL or a non-positive value, the error "Infant travellers passed as negative number or NULL value" is raised.
- If the number of available infant seats is less than or equal to 0, the error "Infant cap reached. Infant cannot be added to the flight" is raised.
- If the infant inventory update fails (i.e., no rows are updated), the error "Infant Inventory update has been failed!" is raised.

## Performance Considerations
- The `UPDATE` operation uses the `WITH (NOLOCK)` hint, which allows reading the data without obtaining an exclusive lock, improving performance by avoiding blocking issues. However, it can also lead to reading uncommitted data.
- The procedure includes early exit checks to prevent unnecessary updates if the input parameters are invalid or if there are no available infant seats.
- Using `SET NOCOUNT ON` eliminates the sending of DONE_IN_PROC messages to the client for each statement, which can slightly enhance performance.

## Version History
- **2020-09-14**:
  - Initial creation by B2B Team.

This document provides a comprehensive overview of the `Flight_SetInfantSeatsSold` stored procedure, detailing its purpose, input parameters, internal logic, dependencies, example usage, error handling, and performance considerations.
