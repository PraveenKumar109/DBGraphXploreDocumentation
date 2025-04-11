---
title: EuropCar_Load
weight: 20
---
## Procedure Overview
The stored procedure `EuropCar_Load` is designed to load Europcar station details, airport station mapping, and ACRISS grid data from CSV files into the respective temporary and main tables in the database. This procedure is primarily used to update the Europcar tables with the latest data from external CSV files, ensuring the database remains current. This loading procedure is essential for maintaining the accuracy and completeness of Europcar station details and associated car rental information.

## Input Parameters
1. `@StationDetailsPath` (NVARCHAR(512)): The file path to the `Station_details_csvformat.csv` file. This file must contain details about the Europcar stations.
   - Constraints: Must not be NULL.

2. `@AirportStationPath` (NVARCHAR(512)): The file path to the `Airport_station_mapping_csvformat.csv` file. This file maps airport codes to station codes.
   - Constraints: Must not be NULL.

3. `@AcrissGridPath` (NVARCHAR(512)): The file path to the `Acriss_grid_cvsformat.csv` file. This file contains ACRISS codes and additional related information.
   - Constraints: Must not be NULL.

## Output Parameters
None.

## Logic Description
1. **Parameter Validation:**
   - The procedure first checks whether any of the input parameters are NULL. If any parameter is NULL, it raises an error and returns an error code `-9`.

2. **File Import:**
   - The procedure uses the `BULK INSERT` command to import data from the specified CSV files into temporary tables.
   - For each file, the row count is checked to ensure data import is successful.
   - If the import fails or no rows are imported, an error message is raised.

3. **Dynamic Table Creation:**
   - Temporary tables `#tmp_station_details`, `#tmp_Airport_station`, and `#tmp_Acriss_grid` are created to hold the imported data.
   - For the `#tmp_Acriss_grid` table, column definitions are dynamically determined based on the header row of the CSV file.

4. **Data Processing and Transformation:**
   - After importing data into temporary tables, the data is processed and transformed:
     - `#station_details` is created to update and store station details with new column definitions and additional columns.
     - Data is updated and consolidated by joining with existing tables such as `EuropCarStation` and `EuropCarStationCarSIPPAssoc`.

5. **Error Handling and Transactions:**
   - The procedure uses transactions to ensure data integrity. 
   - If any error occurs during data insertion or updating, the transaction is rolled back, and an error is raised.

6. **Data Insertion:**
   - Final data is inserted into primary tables `EuropCarStation` and `EuropCarStationCarSIPPAssoc` from their respective temporary tables.
   - Proper handling of foreign keys and ensuring data consistency by deleting and re-inserting with updated values.

## Dependencies
- **Tables:**
  - `EuropCarStation`: Main table storing station details.
  - `EuropCarStationCarSIPPAssoc`: Table holding car SIPP association data.

- **Stored Procedures:**
  - `PrimaryKeyNextValue_GetNextValue`: Generates the next value for primary keys.

## Example Usage
```sql
-- Example call to execute the stored procedure
EXEC EuropCar_Load 
  'C:\CSVtest\Station_details_csvformat.csv', 
  'C:\CSVtest\Airport_station_mapping_csvformat.csv', 
  'C:\CSVtest\Acriss_grid_cvsformat.csv';
```

## Error Handling
- Checks for NULL parameters.
- Uses RAISERROR to handle import and processing errors.
- Transaction management: 
  - Errors within transactions result in rollbacks and logging of the error.
- Specific checks for empty or improperly formatted CSV files.
- Dynamic SQL execution errors are caught, and appropriate error messages are raised.

## Performance Considerations
- Bulk Insert operations for efficient data loading.
- Use of temporary tables to minimize locking and improve transaction performance.
- Dynamic SQL to handle flexible column definitions in the `Acriss` grid.
- Indexes and joins are optimized within temporary tables before final insertion.
- Batch processing within loops ensures large data sets do not cause memory overflow or timeouts.

## Version History
- **Created By**: Vivek Sood
- **Created Date**: 20-Sep-07
- **Purpose**: To upload the Europcar details from the provided CSV files.

Please ensure all file paths are correctly set and verify the data format before executing this procedure for accurate results.