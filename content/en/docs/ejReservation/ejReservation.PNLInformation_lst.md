---
title: PNLInformation_lst
weight: 20
---

## Procedure Overview

### Purpose
The stored procedure `[dbo].[PNLInformation_lst]` is designed to fetch passenger manifest details for specified flights within a given date range. It also accounts for passenger types such as duty travel, staff travel, and standby. The procedure is highly used in generating detailed passenger lists for flights, allowing airlines to have up-to-date and comprehensive information about their passengers.

### Functionality
The procedure retrieves information about passengers including personal details, contact information, travel document details, seat assignments, and travel flags. It processes the data based on the input parameters and returns a result set consisting of passenger information for flights that match the criteria.

### Business Use Case
This stored procedure is primarily used in airline reporting to generate passenger manifests. It is critical for operational efficiency, ensuring that all required passenger details are accessible for flight preparations, compliance, and customer service purposes.

## Input Parameters

| Parameter      | Data Type | Purpose                                       | Expected Values                                 |
|----------------|-----------|-----------------------------------------------|-------------------------------------------------|
| @FlightNumber  | VARCHAR(10)   | The flight number for which passenger details are to be fetched. | Any valid flight number (e.g., 'AB123')          |
| @StartDate     | DATETIME   | The start date of the date range for fetching passenger details. | Any valid date and time within the desired range |
| @EndDate       | DATETIME   | The end date of the date range for fetching passenger details. | Any valid date and time within the desired range |
| @Sector        | VARCHAR(6) | Output parameter that will be set within the procedure to output the flight sector information. | Any valid sector value (e.g., 'Sector1')         |

## Output Parameters

| Parameter      | Data Type  | Significance                                    |
|----------------|------------|-------------------------------------------------|
| @Sector        | VARCHAR(6) | Holds the flight sector information retrieved during the procedure execution. |

## Logic Description

### Step-by-Step Explanation

1. **Initial Configuration:**
    - The `NOCOUNT` is set to `ON` to prevent the sending of row count messages to the client.

2. **Duty Travel Credit Accounts Table Creation:**
    - A temporary table `#DutyTravelCreditAccounts` is created to store credit account IDs with a Duty Travel Flag set to 'Y'.
      ```sql
      CREATE TABLE #DutyTravelCreditAccounts (
          CreditAccountID INT,
          DutyTravelFlag CHAR(1)
      );
      ```

3. **Inserting Duty Travel Accounts:**
    - The `CreditAccount` table is queried to get accounts with Duty Travel Flag as 'Y' and the results are inserted into the temporary table.
      ```sql
      INSERT INTO #DutyTravelCreditAccounts (CreditAccountID, DutyTravelFlag)
      SELECT CreditAccountID, DutyTravelFlag
      FROM CreditAccount WITH (NOLOCK)
      WHERE DutyTravelFlag = 'Y';
      ```

4. **Setting Sector:**
    - The `Sector` output parameter is set by selecting the appropriate substring from the `FlightKey` where the flight number and date range match the input parameters.
      ```sql
      SELECT TOP (1) @Sector = SUBSTRING(FlightKey, 9, 6)
      FROM ejFlight.dbo.Flight WITH (NOLOCK)
      WHERE FlightNumber = @FlightNumber
        AND CONVERT(DATETIME, CONVERT(VARCHAR(12), localdepdttm), 103) >= @StartDate
        AND CONVERT(DATETIME, CONVERT(VARCHAR(12), localdepdttm), 103) <= @EndDate
      ORDER BY LocaldepDt DESC;
      ```

5. **Passenger Information Retrieval:**
    - Information is retrieved from multiple tables using `UNION ALL` to combine results. These tables include `Flight`, `TravelerItinerary`, `PersonName`, `TitleTypeCode`, `Reservation`, and various contact and additional info tables. The query retrieves passenger details and includes additional flags like `Staff Travel Flag`, `Stand By Flag`, and `Duty Travel Flag`.

6. **Combining and Ordering Results:**
    - Data is combined using `UNION ALL` for main passengers, infants, and standby passengers. The results are then ordered by departure date and passenger names.
      ```sql
      ORDER BY Report.localdepdttm, Report.[First Name], Report.[Surname];
      ```

7. **Return Statement:**
    - The procedure ends with a `RETURN 0` indicating successful execution.

## Dependencies

### Tables and Views Referenced

- **Tables:**
  - `CreditAccount`: Stores information regarding credit accounts associated with passengers.
  - `ejFlight.dbo.Flight`: Contains details about flights, including flight number, departure time, and flight key.
  - `TravelerItinerary`, `StandByItinerary`: Stores itinerary information for travelers.
  - `PersonName`, `Infantname`: Holds personal information data for passengers.
  - `TitleTypeCode`: Contains title codes for passenger names.
  - `reservation`: Holds reservation data.
  - `Contact`, `ContactPhone`: Stores contact information for passengers.
  - `PersonAdditionalInfoDocs`: Contains additional documents and personal information for passengers.

### Role of Dependencies
These dependencies are essential for gathering comprehensive data about passengers, including personal information, travel details, contact information, and special travel flags. This ensures that the manifest report generated is complete and accurate.

## Example Usage

### Sample SQL Script

```sql
DECLARE @Sector VARCHAR(6);

EXEC [dbo].[PNLInformation_lst]
    @FlightNumber = 'AB123',
    @StartDate = '2023-09-01 00:00:00',
    @EndDate = '2023-09-30 23:59:59',
    @Sector = @Sector OUTPUT;

SELECT @Sector AS Sector;
```

### Expected Results
The above script will execute the stored procedure for flight number 'AB123' within the specified date range. It will output the flight `Sector` along with passenger details fitting the specified criteria.

## Error Handling

### TRY-CATCH Blocks and Error Codes
- The procedure does not currently use `TRY-CATCH` blocks for error handling.
- Errors are implicitly managed by the SQL Server engine, and the procedure will terminate upon encountering any errors, reverting any partial changes.

### Recommendations for Error Handling
- Implement `TRY-CATCH` blocks to catch and manage SQL errors gracefully.
- Log errors to a dedicated error log table for troubleshooting and audit purposes.

## Performance Considerations

### Optimization Techniques
- **NOLOCK Hint:** `WITH (NOLOCK)` is used to prevent locking and blocking issues during the read operations.
- **Temporary Table:** A temporary table is used to store intermediate results which can reduce re-computation and optimize querying.

### Potential Bottlenecks
- **Large Data Volumes:** Handling large volumes of passenger data and multiple joins across tables can be performance-intensive.
- **Date Conversion:** Frequent use of `CONVERT` for date fields can impact performance.

### Recommendations
- **Indexing:** Ensure proper indexing on the involved tables, especially on columns used in joins and where conditions.
- **Parameter Sniffing:** Use `OPTION(RECOMPILE)` if parameter sniffing causes performance degradation.
- **Query Optimization:** Review and optimize individual queries within the procedure for performance efficiency.

## Version History

### Changes and Updates

| Date        | Description                                            | Modified By     |
|-------------|--------------------------------------------------------|-----------------|
| 12-Sep-2014 | Added fields related to R94 Passenger Manifest Format  | Akanksha Singh  |
| 16-Sep-2014 | Modified to include duty travel flag check             | Akanksha Singh  |
| 22-Sep-2014 | Added Sector parameter for output                      | Akanksha Singh  |


