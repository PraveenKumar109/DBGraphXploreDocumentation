---
title: udf_LocalTime
weight: 20
---
### Procedure Overview
The purpose of the stored procedure `dbo.udf_LocalTime` is to convert a given system datetime to the local datetime for a specified airport based on the GMT offset value of that airport. This is useful in scenarios such as flight scheduling, where it is necessary to present times in the local time zones of different airports.

### Input Parameters
The stored procedure accepts two input parameters:

1. **@AirportCode**:
   - **Data Type**: AirportCodeDesc (assumed to be a user-defined data type representing airport codes)
   - **Purpose**: Identifies the airport for which the local time needs to be calculated.
   - **Expected Values/Constraints**: Must be a valid airport code present in the `dbo.AirportCode` table.

2. **@SystemDtTm**:
   - **Data Type**: datetime
   - **Purpose**: Represents the system date and time that needs to be converted to local time.
   - **Expected Values/Constraints**: Valid datetime value.

### Output
The stored procedure returns a single value:
- **Output Data Type**: datetime
- **Significance**: The local date and time for the specified airport.

### Logic Description
1. **Declare Local Variables**:
   - `@LocalDateTime` (datetime): To store the resulting local datetime.
   - `@GMTOffsetMinutes` (int): To store the GMT offset in minutes for the specified airport.

2. **Fetch GMT Offset**:
   - Query the `dbo.AirportCode` table to retrieve the `WinterGMTOffset` for the given `@AirportCode`.
   - Convert the `WinterGMTOffset` to minutes and reverse the sign (as the offset is stored in an inverse manner).

3. **Calculate Local Time**:
   - Use the `DATEADD` function to add the `@GMTOffsetMinutes` to `@SystemDtTm`.

4. **Return Local Time**:
   - Return the computed local datetime.

### Dependencies
- **Tables**:
  - `dbo.AirportCode`: The procedure queries this table to fetch the `WinterGMTOffset` for the given airport code.

### Example Usage
```sql
DECLARE @AirportCode AirportCodeDesc = 'LHR'
DECLARE @SystemDtTm datetime = '2023-10-01 12:00:00'
DECLARE @LocalTime datetime

SELECT @LocalTime = dbo.udf_LocalTime(@AirportCode, @SystemDtTm)

PRINT @LocalTime -- Outputs the local time for airport 'LHR' based on the provided system datetime.
```

### Error Handling
- The function does not contain explicit error handling such as TRY-CATCH blocks.
- If the `@AirportCode` does not exist in the `dbo.AirportCode` table, the function will return the input `@SystemDtTm` without any offset adjustment, likely resulting in an incorrect local time.

### Performance Considerations
- **WITH (NOLOCK)**: The use of NOLOCK hint ensures that the function operates with minimal locking, enhancing performance. However, it may return dirty reads.
- **Index Consideration**: Ensuring that the `AirportCode` column in the `dbo.AirportCode` table is indexed will improve the performance of the SELECT query used to fetch the `WinterGMTOffset`.

### Version History
- **$Header: /EasyJet/DevCode/Databases/ejFlight/UserDefinedFunctions/udf_LocalTime.UDF 2 3/16/04 12:52p Pnuttall $**
- Initial version created by Pnuttall on 3/16/04.

### Recommendations for Future Enhancements
- **Error Handling**: Incorporate TRY-CATCH blocks to handle unexpected errors gracefully.
- **Offset Adjustments**: Consider handling both winter and summer (daylight saving) offsets if applicable.

This concludes the comprehensive technical documentation for the `dbo.udf_LocalTime` SQL stored procedure.