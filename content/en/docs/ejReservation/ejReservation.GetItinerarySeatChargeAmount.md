---
title: GetItinerarySeatChargeAmount
weight: 20
---

## 1. Procedure Overview

**Purpose:**  
The stored procedure `[dbo].GetItinerarySeatChargeAmount` calculates the total seat charge amount for an itinerary based on a given booking reference number. The procedure is used to fetch itineraries and their respective charges, helping in financial and operational analysis.

**Functionality and Business Use Case:**  
This procedure serves the business need of aggregating seat charges for different itineraries linked to a reservation. It is particularly useful in scenarios where financial reporting on seat charges is required, including differentiating for various charge codes such as specific seat bands or overwing extra legroom charges.

## 2. Input Parameters

- **@BookingReferenceNumber** (NVARCHAR(50)):  
  - **Purpose:** This parameter represents the unique booking reference number used to identify a reservation.
  - **Expected Values:** A valid booking reference code. The value should be string type with a maximum length of 50 characters.

## 3. Output Parameters

This stored procedure does not explicitly define output parameters but returns results as an XML formatted raw data set.

## 4. Logic Description

The logic flow within the stored procedure `[dbo].GetItinerarySeatChargeAmount` can be described step-by-step as follows:

1. **Disable row count messages:**  
   `SET NOCOUNT ON` prevents extra result sets from interfering with SELECT statements.

2. **Declaration of Variables:**
    - Declare `@ReservationID` to hold the relevant reservation ID corresponding to the provided Booking Reference Number.

3. **Fetch Reservation ID:**
    - Retrieve the `ReservationID` from the `dbo.Reservation` table, using the input `@BookingReferenceNumber`.

4. **Calculate Seat Charges:**
    - Select the TravelerItineraryID and the sum of charge amounts from `ejReservation.dbo.ReservationCharges` that are linked to the `ReservationID`.
    - Join with `ejReservation.dbo.TravelerItinerary` based on `TravelerItineraryID` and apply conditions ensuring the itinerary status code equals 1 (active or valid itineraries).
    - Perform a LEFT JOIN with `ejReservation..SeatBand` to include only relevant seat band charges or specific charge code ID (164, denoted as overwing extra legroom for FY19).

5. **Grouping and Filtering:**
    - Group the results by `rc.TravelerItineraryID` and filter using HAVING clause to include only itineraries where the sum of `ChargeAmount` is greater than 0.

6. **Output as XML:**
    - Return the results in an XML RAW format.

## 5. Dependencies

- **Tables Referenced:**
  - `dbo.Reservation`: Used to fetch the ReservationID using the booking reference number.
  - `ejReservation.dbo.ReservationCharges`: Contains the reservation charges data.
  - `ejReservation.dbo.TravelerItinerary`: Stores information regarding the traveler itineraries.
  - `ejReservation..SeatBand`: Used to filter specific seat band related charges.

## 6. Example Usage

```sql
-- Sample call to the stored procedure
DECLARE @BookingReferenceNumber NVARCHAR(50)
SET @BookingReferenceNumber = 'ABCD1234'

EXEC [dbo].GetItinerarySeatChargeAmount @BookingReferenceNumber
```

**Expected Result:**  
XML formatted response with `TravelerItineraryID` and the summed `ChargeAmount` for valid itineraries meeting the provided conditions.

## 7. Error Handling

The procedure currently does not include explicit error handling mechanisms such as TRY-CATCH blocks. Any errors during execution (e.g., invalid booking reference) will result in a system error being thrown.

## 8. Performance Considerations

- **Usage of NOLOCK:**  
  The procedure uses NOLOCK hints to avoid blocking issues, though this may introduce dirty reads. This is a consideration for read-consistency versus performance trade-offs.

- **Indexes:**  
  Ensure that appropriate indexes exist on `Reservation.bookingnumber`, `ReservationCharges.ReservationID`, and other involved columns to optimize query performance.

- **Bottlenecks:**  
  Potential bottlenecks could arise from large data volumes in the `ReservationCharges` table, affecting SUM and GROUP BY operations.

## 9. Version History (Optional)

- **Version 1.0:**  
  - Initial creation of the stored procedure.  
  - Date: [Insert Date]

By documenting in this structured format, the stored procedure’s functionality, usage, and considerations are clearly conveyed, facilitating both development and maintenance efforts.