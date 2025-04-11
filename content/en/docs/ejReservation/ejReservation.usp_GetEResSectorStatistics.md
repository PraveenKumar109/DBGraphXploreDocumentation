---
title: usp_GetEResSectorStatistics
weight: 20
---


## Procedure Overview
The `usp_GetEResSectorStatistics` stored procedure is designed to generate various statistical data related to airline flights for a specified month. This data includes the number of passengers flown, number of no-show passengers, number of flights flown, and flight capacity for specific sectors. The primary business use case it addresses is to provide historical data for flight operations analysis and reporting.

## Input Parameters
The stored procedure accepts one input parameter:

- **@dtMonthStartDate** (datetime): This parameter specifies the start date of the month for which the statistics need to be calculated. It is usually set to the first day of the previous month.

  **Expected Value/Constraint**:
  - Must be a valid `datetime` value.
  - Typically, the value is calculated using: `DATEADD(mm, DATEDIFF(mm,0,GETDATE())-1, 0)` to get the first day of the previous month.

## Output Parameters
There are no output parameters for this stored procedure. It returns a result set with the following columns:
- YearMonth (varchar(6)): The year and month in the format `YYYYMM`.
- Sector (varchar(6)): The flight sector code.
- EquipmentID (int): The equipment ID of the flight (only applicable for the "Number of Flights Flown" statistic).
- StatisticsType (varchar(50)): The type of statistics ('NumberOfPassengersFlown', 'NumberOfPassengersNoShow', 'NumberOfFlightsFlown', 'Capacity').
- Value (decimal(28,8)): The calculated statistical value.

## Logic Description
1. **Initial Setup**:
   - The procedure begins by disabling row count messages (`SET NOCOUNT ON;`) and setting ANSI_NULLS to OFF.
   - Two TRY-CATCH blocks are utilized to handle errors.
   - The `@vchProcedureName` variable is declared and set to the name of this stored procedure.
   - Error handling variables (`@ErrorMessage`, `@ErrorSeverity`, `@ErrorState`, and `@ErrorNumber`) are initialized.
   - The input date (`@dtMonthStartDate`) is used to calculate the starting and ending dates for the month (`@dtFromDate` and `@dtToDate`).

2. **Number Of Passengers Flown**:
   - A query is executed to calculate the total number of passengers who have flown (checked-in passengers) per sector. The results are grouped by sector and month.

3. **Number Of Passengers No Show**:
   - Another query calculates the number of passengers who did not show up by checking if their `CheckedInFlag` is set to 'N' or NULL. The results are grouped by sector and month.

4. **Number Of Flights Flown**:
   - This section counts the number of distinct flights flown per sector and equipment ID, provided the flight has at least one checked-in passenger.

5. **Capacity**:
   - The actual capacity is calculated by multiplying the number of flights flown by the average seats sold.

6. **Error Handling**:
   - If an error occurs, the error details are captured using `ERROR_NUMBER()`, `ERROR_MESSAGE()`, `ERROR_SEVERITY()`, and `ERROR_STATE()`, and a custom error message is raised.

## Dependencies
This stored procedure references and interacts with the following database objects:
- **Tables**:
  - `ejFlight.dbo.Flight`: Contains information about flights.
  - `ejReservation.dbo.TravelerItinerary`: Contains traveler itinerary details.
  - `ejFlight.dbo.RouteData`: Contains route-related data.

  These tables are involved in joins and data retrieval operations to calculate the required statistics.

## Example Usage
Here is an example SQL script demonstrating how to call the procedure:

```sql
-- Example: Calling the procedure for the previous month
DECLARE @StartOfMonth datetime;
SET @StartOfMonth = DATEADD(mm, DATEDIFF(mm,0,GETDATE())-1, 0);
EXEC dbo.usp_GetEResSectorStatistics @dtMonthStartDate = @StartOfMonth;
```

## Error Handling
The stored procedure employs a TRY-CATCH block to handle exceptions. If an error occurs, it:
- Retrieves the error details using system functions such as `ERROR_NUMBER()`, `ERROR_MESSAGE()`, `ERROR_SEVERITY()`, and `ERROR_STATE()`.
- Constructs a custom error message prepended with the procedure name.
- Raises the error using the `RAISERROR` function with the captured error details.
- Returns the error number to the calling environment.

## Performance Considerations
- The procedure uses `WITH (NOLOCK)` hints to avoid locking issues and improve performance when reading from the tables. However, this may lead to reading uncommitted data.
- The procedure is optimized by using efficient date calculations to determine the start and end dates of the month.
- Indexes on the `flightid` and `Sector` columns in the referenced tables can improve join and grouping operations.
- Aggregations and data retrieval are performed in a single pass to minimize data processing overhead.

## Version History
- **27 Dec 2007**: Initial version created by Rich Hawkins.
- **28-Apr-2008**: Added `PlaneType`.
- **27-Jun-2008**: Added `Capacity`.
- **21-Jul-2008**: Amended capacity calculations.
- **13-Jan-2009**: Modified by Andrew Nuttall to run on eRes LIVE.