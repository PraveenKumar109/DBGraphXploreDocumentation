---
title: Reservation_GetClassification
weight: 20
---

## Procedure Overview:

**Procedure Name:** `[dbo].[Reservation_GetClassification]`

**Purpose:** 
The `Reservation_GetClassification` stored procedure is designed to classify and return various attributes related to a reservation based on a given booking reference. These attributes include disruption status, baggage counts, traveler details, special service requirements, and other related information. This procedure helps to quickly ascertain the reservation's current state and associated services, which is essential for operational insights and customer service.

**Business Use Case:**
This stored procedure supports the reservations system by providing a detailed classification of booking attributes, which can be used by airlines and travel agencies to manage flight logistics, customer service, and resource allocation efficiently. It ensures that relevant information about the reservation is readily available, which is critical for decision-making and enhancing customer experience.

## Input Parameters:

1. **`@BookingReference`**
   - **Data Type:** `VARCHAR(20)`
   - **Purpose:** This parameter is the booking reference number used to identify the specific reservation.
   - **Expected Values/Constraints:** It should be a valid booking reference string, typically assigned during the booking process.

## Output Parameters:

There are no explicitly defined output parameters, but the stored procedure returns a result set with the classification details of the reservation.

## Logic Description:

1. **Initialization:**
   - Several local variables are declared and initialized to hold various attributes and status counts related to the reservation.

2. **Booking Number Assignment:**
   - The input booking reference is assigned to the `@BookingNumber` variable.

3. **Reservation ID Retrieval:**
   - The procedure queries the `dbo.Reservation` table to retrieve the `ReservationID` using the booking reference.
   - If not found, it tries using the GDS booking reference.

4. **Error Handling:**
   - If the `ReservationID` is still not found, an error is raised, and the procedure returns -9.

5. **Disruption Status Check:**
   - Checks if any flights in the reservation are disrupted using the `ItineraryInfoView` view.

6. **Bags and Sports Goods Count:**
   - The maximum number of purchased bags and total count of sports goods are retrieved from the `dbo.FlightCheckinBagCounts` table.

7. **Duty Travel Payment Check:**
   - Checks if it's a duty travel booking by joining `Payment` and `CreditAccount` tables.

8. **Passenger Count:**
   - Counts the number of adults, children, and infants from the `TitleTypeCode` and `PersonName` tables.

9. **Object on Seat Check:**
   - Checks if there are any objects on seats using the `PersonName` table with a specific `TitleTypeCodeId`.

10. **Membership Check:**
    - Validates if the booking includes a member from the `memberprofile` and `MembershipReservationAssoc` tables.

11. **Seat Status and Flight Count:**
    - Retrieves various seat status counts and the total number of flights from the `TravelerItinerary` table.

12. **Special Service Requirement (SSR) Count:**
    - Counts the special service requirements from `TravelerItinerary` and `SpecialServiceRequestCode` tables.

13. **Corporate Discount Check:**
    - Checks for corporate discounts using the `dbo.CorporateReservationAssoc` table.

14. **Ancillary Product Counts:**
    - Counts travel insurance and car hire ancillary products from the `AncillaryProductUsage` table.

15. **Inflight Voucher Purchase Check:**
    - Calculates the total cost of inflight vouchers from the `ReservationCharges` and `ChargeCode` tables.

16. **Divergent Flights Check:**
    - Checks if there are divergent flights, meaning any passenger does not have all flights using the `TravelerItinerary` table.

17. **Standby Passenger Check:**
    - Counts standby passengers from the `StandbyItinerary` table.

18. **Booking Version Type:**
    - Retrieves the booking version from the `Reservation` table.

19. **Closed Flights Count:**
    - Counts the number of closed flights from various tables including `ejFlight.dbo.Flight` and `ejFlight.dbo.AirportCode`.

20. **Next Flight ID Retrieval:**
    - Retrieves the next flight ID by joining `ejFlight.dbo.Flight` and `TravelerItinerary` tables.

21. **VT100 Booking Check:**
    - If no member is found, the booking is classified as VT100.

22. **Result Compilation:**
    - Formats and selects the data into a final result set for output.

## Dependencies:

- **Tables:**
  - `dbo.Reservation`
  - `dbo.FlightCheckinBagCounts`
  - `Payment`
  - `CreditAccount`
  - `TitleTypeCode`
  - `PersonName`
  - `ItineraryInfoView`
  - `TravelerItinerary`
  - `SpecialServiceRequestCode`
  - `TravelerItineraryPersonNameSSRAssoc`
  - `dbo.CorporateReservationAssoc`
  - `AncillaryProductUsage`
  - `ReservationCharges`
  - `StandbyItinerary`
  - `ejFlight.dbo.Flight`
  - `ejFlight.dbo.AirportCode`

- **Views:**
  - `ItineraryInfoView`

## Example Usage:

```sql
-- Example of calling the stored procedure:
EXEC [dbo].[Reservation_GetClassification] @BookingReference = 'BR1234567890';
```

## Error Handling:

The procedure primarily uses the `RAISERROR` function to handle scenarios where the `ReservationID` cannot be found. It raises a custom error with a specific error code (50010) and a message indicating that the booking number or reservation ID must be specified. 

Here is the error handling block:
```sql
IF @ReservationId = 0
BEGIN
    RAISERROR(
        50010,
        11,
        1,
        @ProcedureName
    );
    RETURN -9;
END
```

## Performance Considerations:

- **NOLOCK Hints:** The stored procedure uses `WITH (NOLOCK)` hints extensively to avoid locking issues and improve performance, especially in read-heavy environments.
- **Index Usage:** Proper indexing on columns used in the `WHERE` clauses (such as `ReservationID`, `BookingNumber`) is crucial for optimal performance.
- **Transaction Management:** There are no explicit transactions in this procedure, as it primarily performs read operations.
- **Efficient Data Retrieval:** The use of aggregate functions like `SUM` and `MAX` helps in minimizing the result set size and improving retrieval speed.
- **Limiting Data Scope:** Queries are well-scoped to ensure only necessary data is fetched and processed.

## Version History:

**Version:** 1.0
- **Date:** October 2023
- **Changes:** Initial creation and implementation of the `Reservation_GetClassification` stored procedure.

---

This technical document provides a comprehensive overview and detailed explanation of the `Reservation_GetClassification` stored procedure. It covers the purpose, logic, usage, dependencies, and performance considerations crucial for understanding and effectively using the procedure.