---
title: EuropCarStation_upd
weight: 20
---

## Procedure Overview:
The `dbo.EuropCarStation_upd` stored procedure is designed to update the details of an existing EuropCar station in the database. The procedure evaluates the presence of a station by its `StationID` and updates its attributes with the provided input values. This procedure addresses the need to maintain current and accurate station details, which is crucial for operations related to station management.

## Input Parameters:
The procedure accepts the following input parameters:

| Parameter Name         | Data Type   | Purpose                                                    | Expected Values or Constraints              |
|------------------------|-------------|------------------------------------------------------------|---------------------------------------------|
| `@StationID`           | `int`       | Unique identifier of the EuropCar station                  | Mandatory. Integer representing the station ID.|
| `@StationCode`         | `char(6)`   | Code of the EuropCar station                               | Optional. Up to 6 character code.           |
| `@PrimaryStationFlag`  | `char(1)`   | Indicates if it is a primary station; 'Y' or 'N'           | Optional. 'Y' or 'N'                        |
| `@ActiveFlag`          | `char(1)`   | Indicates if the station is active; 'Y' or 'N'             | Optional. 'Y' or 'N'                        |
| `@AirportCode`         | `char(3)`   | Code of the airport where the station is located           | Mandatory. 3 character code such as IATA code.|
| `@StationName`         | `varchar(100)` | Name of the EuropCar station                              | Optional. Up to 100 characters.             |
| `@StationAddress`      | `varchar(1000)` | Address of the EuropCar station                          | Optional. Up to 1000 characters.            |
| `@CountryCode`         | `char(2)`   | Country code                                               | Optional. 2 character country code.         |
| `@StationPhone`        | `varchar(30)` | Phone number of the EuropCar station                      | Optional. Up to 30 characters.              |
| `@StationFax`          | `varchar(30)` | Fax number of the EuropCar station                        | Optional. Up to 30 characters.              |
| `@HoursOfOperation`    | `varchar(500)` | Hours of operation of the EuropCar station               | Optional. Up to 500 characters.             |
| `@EuropCarMapLink`     | `varchar(150)` | URL link to the EuropCar map of the station              | Optional. Up to 150 characters.             |
| `@URLtoTypicalModels`  | `varchar(150)` | URL link to typical models available at the station      | Optional. Up to 150 characters.             |

## Output Parameters:
This stored procedure does not have any output parameters.

## Logic Description:
1. **Initialization**:
   - Turn off the count of affected rows feature using `SET NOCOUNT ON`.
   - Declare local variables `@ErrNum` and `@ProcedureName`.

2. **Procedure Identification**:
   - Retrieve the name of the procedure using `OBJECT_NAME(@@ProcID)`.

3. **Check for Existing Station**:
   - `IF EXISTS` clause checks if a station with the provided `@StationID` exists in the `dbo.EuropCarStation` table.
   
4. **Update Operation**:
   - If the station exists, execute an `UPDATE` statement.
   - Use the `COALESCE` function to update each field with either the provided input parameter or maintain the existing value if the input parameter is `NULL`.

5. **Error Handling for Non-existent Station**:
   - If the station does not exist, raise a custom error using `RAISERROR` with error number `50020`.

6. **Return Statement**:
   - Return `0` to indicate successful completion or `-9` for an error.

## Dependencies:
- **Table**: 
  - `dbo.EuropCarStation`: The table where the station details are stored. The stored procedure reads from and updates this table as needed.

## Example Usage:
```sql
-- Example of how to call the procedure

EXEC dbo.EuropCarStation_upd
    @StationID = 1,
    @StationCode = 'STC001',
    @PrimaryStationFlag = 'Y',
    @ActiveFlag = 'Y',
    @AirportCode = 'JFK',
    @StationName = 'John F. Kennedy EuropCar Station',
    @StationAddress = '123 Main St, New York, NY 10001, USA',
    @CountryCode = 'US',
    @StationPhone = '+1-123-456-7890',
    @StationFax = '+1-123-456-7891',
    @HoursOfOperation = '24/7',
    @EuropCarMapLink = 'http://examplemap.com',
    @URLtoTypicalModels = 'http://examplemodels.com'
```

## Error Handling:
- **Custom Errors**:
  - The procedure uses `RAISERROR` to throw a custom error if the station with the provided `@StationID` does not exist. The error number `50020` is used in this context.

- **Default Values Handling**:
  - The `COALESCE` function is used to handle the default behaviour of keeping existing values if the input parameters are `NULL`.

## Performance Considerations:
- **NOLOCK Hint**: 
  - The `NOLOCK` hint is used in the `SELECT` statement to avoid locking the table, which can improve performance during read operations.

- **COALESCE Function**: 
  - Efficiently handles `NULL` values to avoid unnecessary writes to the database when params are not provided.

- **Error Handling**: 
  - Using `RAISERROR` for clear error messaging while minimizing the procedure's complexity.

## Version History:
- **Initial Version**: 
  - Date: 7/21/05, implemented by Iochman.
