---
title: RouteCategoryDe_Activate
weight: 20
---
## Procedure Overview
The `RouteCategoryDe_Activate` stored procedure is designed to manage the activation and deactivation status of route categories within the `ejFlight` database. This stored procedure allows users to activate an inactive route category or deactivate an active route category, with the additional functionality of confirming the deactivation. This helps in organizing and maintaining the operational route categories efficiently.

## Input Parameters

### @RouteCategoryID
- **Data Type**: `EntityIdentifier` (Assumed to be an alias for a data type, typically an integer or unique identifier)
- **Purpose**: Identifies the route category to be activated or deactivated.
- **Expected Values or Constraints**: Should be a valid and existing RouteCategoryID within the `RouteCategory` table.

### @ConfirmDeactivation
- **Data Type**: `BIT`
- **Purpose**: Confirms whether the deactivation should proceed with all associated data deletions and updates.
- **Expected Values or Constraints**: `1` (True) to confirm deactivation and associated actions; `NULL` or `0` (False) to check for existing associations before deactivating without deleting related data.

## Output Parameters
There are no explicit output parameters for this stored procedure.

## Logic Description

1. **Initial Setup**:
    - **NOCOUNT Setting**: `SET NOCOUNT ON` to prevent extra result sets from interfering with queries.
    - **Variable Declaration**: `@Active BIT` to store the current active status of the route category.

2. **Fetch Current Status**:
    - **Query**: Select the `Active` status from `ejFlight.dbo.RouteCategory` using `@RouteCategoryID`.
    - If the route category does not exist (`@Active IS NULL`), raise an error "RouteCategory does not exist".

3. **Activation Logic**:
    - **Check `@Active`**: If `@Active = 0`, update the status to `1` (active).

4. **Deactivation Logic**:
    - If `@Active` is currently `1`:
        - **Confirm Deactivation**: If `@ConfirmDeactivation = 1`:
            - Delete entries from `dbo.RouteCategoryRouteAssoc`.
            - Update `Active` status to `0` in `dbo.RouteCategoryPOSFeeAdjustment`.
            - Deactivate the route category in `ejFlight.dbo.RouteCategory`.
        - **Without Confirmation**: If `@ConfirmDeactivation` is not `1` (null or false):
            - **Check Dependencies**: If active associations exist in `RouteCategoryPOSFeeAdjustment` or `RouteCategoryRouteAssoc`:
                - Raise an error (code `500165`) and return `-9`.
            - **No Dependencies**: Deactivate the route category.

## Dependencies

### Tables
1. **ejFlight.dbo.RouteCategory**
    - Stores route category information, including the `Active` status.
2. **dbo.RouteCategoryRouteAssoc**
    - Stores associations related to route categories.
3. **dbo.RouteCategoryPOSFeeAdjustment**
    - Contains point-of-sale fee adjustment records linked to route categories.

## Example Usage

```sql
-- Example 1: Activate an inactive route category
EXEC dbo.RouteCategoryDe_Activate @RouteCategoryID = 12345;

-- Example 2: Deactivate an active route category without confirmation
EXEC dbo.RouteCategoryDe_Activate @RouteCategoryID = 12345, @ConfirmDeactivation = 0;

-- Example 3: Deactivate an active route category with confirmation
EXEC dbo.RouteCategoryDe_Activate @RouteCategoryID = 12345, @ConfirmDeactivation = 1;
```

## Error Handling

- **Route Category Does Not Exist**:
    - Error Message: `'RouteCategory does not exist'`
    - Error Severity: `16`
    - Error State: `-1`
- **Active Associations Exist**:
    - Error Message Code: `500165`
    - Error Severity: `16`
    - Error State: `-1`
    - Returns `-9` indicating that the deactivation cannot proceed due to existing associations.

## Performance Considerations

- **Indexing**: Ensure that `RouteCategoryID` is indexed in `RouteCategory`, `RouteCategoryRouteAssoc`, and `RouteCategoryPOSFeeAdjustment` to improve query performance.
- **NOLOCK Hints**: Used to prevent locking during the read operations, which can improve performance in high-concurrency environments but may risk reading uncommitted data.
- **Batch Operations**: The deletion and updates are done in batch, reducing the number of context switches and improving performance.

## Version History
- **Version 1.0**: Initial version created.
- **Date**: [Insert Date]
- **Changes**: Initial implementation and documentation.

This comprehensive document should provide a clear understanding of the `RouteCategoryDe_Activate` stored procedure, its functionality, usage, and key considerations for both developers and database administrators.

