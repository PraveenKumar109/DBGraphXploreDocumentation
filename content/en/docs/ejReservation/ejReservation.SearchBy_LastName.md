---
title: SearchBy_LastName
weight: 20
---

## Procedure Overview

The `SearchBy_LastName` stored procedure is designed to search for booking records within the company's reservation system based on a provided last name. It fetches relevant details from multiple related tables to provide a comprehensive overview of bookings, including information such as first name, booking number, reservation ID, departure and arrival airport codes, and booking dates. This stored procedure is especially useful for customer support and booking agents who need to retrieve booking details quickly and efficiently based on customer last names.

## Input Parameters

| Parameter          | Data Type   | Purpose                                                                                      | Constraints                                |
|--------------------|-------------|----------------------------------------------------------------------------------------------|--------------------------------------------|
| `@LastName`        | `VARCHAR`   | The last name of the person whose booking information is to be retrieved.                    | Must be a valid last name string, can include wildcards such as `%` for SQL `LIKE` operation.   |
| `@LimitResultsTo`  | `int`       | An optional parameter to limit the number of results returned by the query.                  | Must be a positive integer or `NULL` if not used. |
| `@VersionNumber`   | `int`       | A version identifier for the stored procedure.                                               | Default value is set to `1`.

## Output Parameters

This stored procedure does not use output parameters but returns a dataset containing booking and travel itinerary details in XML format using the `FOR XML RAW` clause.

## Logic Description

1. **SET NOCOUNT ON**:
    - Prevents the message indicating the number of rows affected by T-SQL statements from being returned.

2. **Initial Commented Out Validation**:
    - A block of code that counts the number of rows matching the last name and raises an error if it exceeds `@LimitResultsTo` is commented out, indicating either it is not currently in use or under consideration for deprecation.

3. **Main Query**:
   - The core logic consists of two primary subqueries combined using `UNION ALL`, each extracting relevant booking information from different sources (`PersonName` and `Contact` tables), joined with the `Reservation` table.
   - These subqueries are then unionized to remove duplicates (distinct is removed due to union usage).
   
4. **Inner Join with TravelerItinerary Table**:
   - The results from the union of the subqueries are joined with the `TravelerItinerary` table to enrich the dataset with travel-specific details such as flight date.

5. **Ordering**:
   - The final result set is ordered by `LocalDepDt` in descending order and `LastName` in ascending order. 

6. **XML Format**:
   - The procedure includes the `FOR XML RAW` clause to return results in a structured XML format, making it easier for further consumption by other systems or applications.

## Dependencies

- **Tables**:
  - `PersonName`: Contains personal details of people related to reservations.
  - `Contact`: Contains contact details related to reservations.
  - `Reservation`: Contains booking details, including reservation ID and booking dates.
  - `TravelerItinerary`: Contains travel itinerary details for a reservation.
  
All tables are accessed with a `NOLOCK` hint to prevent locking issues and improve performance.

## Example Usage

```sql
DECLARE @XMLResult XML

EXEC dbo.SearchBy_LastName
  @LastName = 'Smith%',
  @LimitResultsTo = 100,
  @VersionNumber = 1
```

In this example, the procedure is called to search for all bookings with last names starting with 'Smith' and limiting the results to 100 entries.

## Error Handling

- The commented-out validation block indicates a potential feature to limit the number of records and raise an error if the count exceeds `@LimitResultsTo`, intended to avoid overwhelming users with too many results.
- Currently, there is no explicit `TRY-CATCH` block or other error handling constructs within the procedure.

## Performance Considerations

- Use of the `NOLOCK` hint to prevent locking issues on the `PersonName`, `Contact`, and `Reservation` tables.
- The `UNION ALL` is used instead of `UNION` to avoid the performance cost of duplicate checks, given the use case.
- Indexes on the relevant columns (e.g., `LastName`, `ReservationID`, etc.) are assumed but not explicitly managed within the stored procedure.

## Version History

- **1.0** (07/27/2006) - Initial release by Edmondv with header documentation.
- Modifications:
  - **R97** - Included flight date in return results for `VT100 Option 1`.
  - **Nadeem** - Added `HolidaysBooking` field for holidays in `VT100`.

This document aims to provide all necessary insights required for understanding and effectively utilizing the `SearchBy_LastName` stored procedure. For further inquiries or modifications, please contact the database administrator team.