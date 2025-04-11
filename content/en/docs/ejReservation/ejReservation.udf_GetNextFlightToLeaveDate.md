---
title: udf_GetNextFlightToLeaveDate
weight: 20
---

## 1. Procedure Overview
The SQL function `udf_GetNextFlightToLeaveDate` is designed to retrieve the next scheduled departure date and time for a flight associated with a given reservation. This function precisely aims to provide the earliest departure time of any flight that is associated with a reservation, considering only future departures at the time of the query. This is particularly useful for applications that need to notify travelers or manage schedules based on upcoming flights.

## 2. Input Parameters
### @reservationId
- **Data Type:** INT
- **Purpose:** This parameter is used to specify the unique identifier of the reservation for which the next flight's departure date and time is to be fetched.
- **Expected Values or Constraints:** Must be a valid `ReservationID` present in the `ejReservation.dbo.Reservation` table.

## 3. Output Parameters
### Return Value
- **Data Type:** DATETIME
- **Significance:** The returned value represents the departure date and time of the next flight associated with the given reservation that is scheduled to depart after the current UTC time.

## 4. Logic Description
The function executes the following steps:
1. **Initialization:** Declares a variable `@Result` to store the resulting departure date and time.
2. **Query Execution:**
   - Performs a SQL `SELECT` query to find the minimum (`MIN`) departure date and time (`LocalDepDtTm`) from the `ejFlight.dbo.Flight` table, which is associated with the provided `ReservationID`.
   - This query uses the `INNER JOIN` method to combine the `Reservation`, `TravelerItinerary`, and `Flight` tables based on keys `ReservationID` and `FlightID`.
   - The condition `f.LocalDepDtTm > GETUTCDATE()` ensures that only future flight departures are considered.
3. **Return Result:** The function returns the earliest departure date and time (`@Result`) found.

## 5. Dependencies
**Tables:**
- `ejReservation.dbo.Reservation`: Contains the reservation details.
- `ejReservation.dbo.TravelerItinerary`: Maps travelers to their itineraries.
- `ejFlight.dbo.Flight`: Contains flight schedule details.

**Role Explanation:**
- The `Reservation` table provides the unique reservation records.
- The `TravelerItinerary` table links reservations to specific flights.
- The `Flight` table holds the schedule details needed to identify the next flight.

## 6. Example Usage
Below is a sample SQL script to demonstrate how to call the function:

```sql
DECLARE @NextFlight DATETIME;

-- Call the function with a specific reservation ID
SET @NextFlight = dbo.udf_GetNextFlightToLeaveDate(12345);

-- Output the result
SELECT @NextFlight AS 'Next Flight Departure Date and Time';
```
In this example:
- `12345` is the reservation ID for which the next flight departure date and time is fetched.
- The result is stored in the variable `@NextFlight` and then selected for display.

## 7. Error Handling
The function does not include explicit error handling within its definition. However, the following implicit error considerations apply:
- If the function does not find any matching future flights, it will return `NULL`.
- SQL Server's native error handling for invalid data types and out-of-range values applies.

It is recommended to implement additional error handling mechanisms in the calling application or SQL code to manage potential issues, such as invalid reservation IDs or connection failures.

## 8. Performance Considerations
- **Indexes:** Ensure that `ReservationID` and `FlightID` columns used in joins are indexed for faster performance.
- **NOLOCK Hint:** The use of `(NOLOCK)` minimizes locking and blocking behavior but should be used with caution to avoid dirty reads.
- **Query Efficiency:** The use of `MIN` function combined with filtering on `LocalDepDtTm > GETUTCDATE()` ensures efficient identification of the earliest future flight.

## 9. Version History
- **Initial version:** Implemented on [Date], crafted by [Author Name].

---

This document provides a comprehensive overview of the function `udf_GetNextFlightToLeaveDate`, including its usage, logic, dependencies, and considerations for optimizing its performance.