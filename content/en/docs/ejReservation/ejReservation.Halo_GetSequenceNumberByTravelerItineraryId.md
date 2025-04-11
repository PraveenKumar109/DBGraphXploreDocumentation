---
title: Halo_GetSequenceNumberByTravelerItineraryId
weight: 20
---

## 1. Procedure Overview
The stored procedure `[dbo].[Halo_GetSequenceNumberByTravelerItineraryId]` is designed to retrieve the sequence number of a traveler based on a given itinerary ID. Specifically, it fetches the `BoardingNumber` associated with the `TravelerItineraryId`. If the `BoardingNumber` is not available, it falls back on the `StsNumber`. This procedure is essential for the business process of identifying and managing traveler itineraries efficiently.

## 2. Input Parameters

### @TravelerItineraryId
- **Data Type**: BIGINT
- **Purpose**: This parameter represents the unique identifier for the traveler's itinerary.
- **Expected Values/Constraints**: The value should be a valid `BIGINT` corresponding to an existing `TravelerItineraryId` in the `TravelerItinerary` table.

## 3. Output Parameters
This procedure does not utilize direct output parameters. Instead, it returns a result set with the required sequence number (`BoardingNumber` or `StsNumber`).

## 4. Logic Description

1. **Start of Procedure**: The procedure begins by setting `NOCOUNT ON` to avoid sending extra result sets to the client, which can improve performance.
   
2. **Query Execution**:
    - The procedure performs a `SELECT` statement on the `TravelerItinerary` table with a `(NOLOCK)` hint to avoid locking the table during the read operation.
    - The `WHERE` clause filters the results based on the input `@TravelerItineraryId`.
    - The `SELECT` statement retrieves the `BoardingNumber` column. If `BoardingNumber` is `NULL`, it selects the `StsNumber` as a substitute. If both are `NULL`, it returns an empty string.
   
3. **End of Procedure**: The procedure completes its execution and returns the results to the caller.

## 5. Dependencies

- **Tables**:
  - `dbo.TravelerItinerary`: This table contains details of traveler itineraries, including `BoardingNumber` and `StsNumber`.

## 6. Example Usage

### Sample SQL Script to Call the Procedure

```sql
DECLARE @TravelerItineraryId BIGINT
SET @TravelerItineraryId = 12345 -- Example itinerary ID

EXEC [dbo].[Halo_GetSequenceNumberByTravelerItineraryId] @TravelerItineraryId
```

### Expected Results
If `TravelerItineraryId` 12345 exists and has a `BoardingNumber`, it will return the `BoardingNumber`. If `BoardingNumber` is `NULL`, it will return the `StsNumber`. If both are `NULL`, it will return an empty string.

## 7. Error Handling
This stored procedure does not include explicit error handling mechanisms such as TRY-CATCH blocks. However, implicit error handling will occur:
   - If the given `TravelerItineraryId` does not exist, it will return an empty result set.
   - It's recommended to implement error handling in the calling application to manage such cases effectively.

## 8. Performance Considerations

- **NOLOCK Hint**: The `(NOLOCK)` table hint is used to prevent locking, enhancing read performance by avoiding contention with other transactions.
- **Indexing**: Ensure that the `TravelerItineraryId` column is properly indexed to optimize the `SELECT` query.
- **SET NOCOUNT ON**: This setting helps in reducing network traffic due to the elimination of row count messages.

## 9. Version History (Optional)
- **Version 1.0**: Initial creation - October 1, 2023

*Note: Update this section to include any future modifications or enhancements to the procedure.*

---

This detailed technical document provides a comprehensive understanding of the `Halo_GetSequenceNumberByTravelerItineraryId` stored procedure, facilitating its effective use and maintenance.