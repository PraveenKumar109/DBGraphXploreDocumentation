---
title: flexi_selectedsectornames_get
weight: 20
---

## Procedure Overview
The `flexi_selectedsectornames_get` stored procedure is designed to retrieve a list of concatenated airport codes based on a specified group name. It achieves this by first identifying the group ID associated with the provided group name, then selecting the corresponding departure and arrival airport codes from the `FlexiRouteGroupAssoc` table. This procedure is primarily used in the context of flight reservations and route management.

## Input Parameters
### @GroupName
- **Data Type:** `varchar(50)`
- **Purpose:** The name of the group for which airport codes need to be retrieved.
- **Expected Values or Constraints:** Should be an existing group name in the `FlexiGroup` table. The length of the string should not exceed 50 characters.

## Output Parameters
This stored procedure does not have any output parameters. Instead, it returns a result set of concatenated airport codes.

## Logic Description
1. **Initial Declarations:**
    - Declare a local variable `@Groupid` of type `EntityIdentifier` to store the ID of the group corresponding to the provided `@GroupName`.

2. **Retrieve Group ID:**
    - Execute a `SELECT` query to fetch the `FlexiGroupID` from the `FlexiGroup` table in the `ejReservation` database where the `FlexiGroupName` matches the provided `@GroupName`.
    - Store the retrieved `FlexiGroupID` in the `@Groupid` variable.

3. **Fetch and Concatenate Airport Codes:**
    - Execute a `SELECT` query to fetch concatenated airport codes from the `ejflight.dbo.airportcode` table.
    - Join the `airportcode` table twice using the `depairportcodeid` and `arrairportcodeid` from the `FlexiRouteGroupAssoc` table.
    - Filter the results to include only active records (`ActiveFlag = 'Y'`) and where the `FlexiGroupID` matches the value stored in `@Groupid`.
    - Concatenate the departure and arrival airport codes and order the result set accordingly.

## Dependencies
### Tables
- **`ejReservation.dbo.FlexiGroup`:**
  - Stores the details of various flexi groups.
  - Used to retrieve the `FlexiGroupID` using the provided `@GroupName`.

- **`ejflight.dbo.airportcode`:**
  - Stores airport codes and related information.
  - Utilized to fetch the airport codes that are joined and concatenated.

- **`FlexiRouteGroupAssoc`:**
  - Stores the association between the flexi group, departure airport code, and arrival airport code.
  - Used to filter active routes and associate airport codes for the specified `FlexiGroupID`.

## Example Usage
```sql
-- Example call to the stored procedure
EXEC [dbo].[flexi_selectedsectornames_get] @GroupName = 'Business Travelers';
```

### Expected Result
```plaintext
JFKLHR
LAXSFO
SFOORD
...
```
(The exact result will depend on the data present in the database.)

## Error Handling
This procedure does not involve explicit error handling mechanisms within its body (no `TRY-CATCH` blocks). However, it relies on standard SQL error reporting. Potential errors could occur if:
- The provided `@GroupName` does not exist in the `FlexiGroup` table.
- There are database connectivity issues.

## Performance Considerations
- **Index Usage:** Ensure that the `FlexiGroup`, `airportcode`, and `FlexiRouteGroupAssoc` tables are properly indexed on the columns used in the joins and conditions to enhance performance.
- **NOLOCK Hint:** The `WITH(NOLOCK)` hint is used to reduce locking contention and improve query performance. However, it may lead to reading uncommitted data.
- **Execution Plan:** Regularly analyze the execution plan and optimize if there are changes in data volume or query performance degrades.

## Version History
- **Initial Version:**
  - **Date:** [Current Date]
  - **Description:** Initial creation of the `flexi_selectedsectornames_get` stored procedure to retrieve concatenated airport codes based on a group name parameter.

---
This technical document provides a comprehensive overview of the `flexi_selectedsectornames_get` stored procedure, describing its functionality, usage, and additional considerations for effective deployment and maintenance.