---
title: Contact_get
weight: 20
---
## Procedure Overview
The `[dbo].[Contact_get]` stored procedure is designed to retrieve contact details from the `Contact` table in the `ejReservation` database. This procedure fetches and returns all relevant contact information associated with a specific `ReservationID`. The primary business use case for this procedure is to provide detailed contact information based on a reservation identifier, essential for customer service, reporting, and reservation management tasks.

## Input Parameters
The stored procedure takes the following input parameter:

- **`@ReservationID`**: 
  - **Data Type**: `EntityIdentifier` (typically an integer or unique identifier depending on the table schema)
  - **Purpose**: This parameter specifies the unique identifier of the reservation for which contact details are to be fetched.
  - **Expected Values or Constraints**: `@ReservationID` should be a non-null value representing a valid reservation identifier.

## Output Parameters
There are no explicit output parameters. However, the procedure returns a result set containing the contact details if the `ReservationID` is valid.

## Logic Description
The logic within the stored procedure is structured as follows:

1. **SET NOCOUNT ON**: This ensures that the count of rows affected by a SQL statement is not returned.
2. **Input Validation**: 
    - Checks if the input parameter `@ReservationID` is not null.
3. **Data Selection**:
    - If `@ReservationID` is valid, it selects contact details from the `Contact` table with `NOLOCK` hint.
    - Fields selected include `ReservationID`, `Address1`, `Address2`, `City`, `StateProvince`, `PostalCode`, `Country`, `EmailAddress`, `CustomerEmailAddress`, `TicketOptionCodeID`, `RowStatusCode`, `FirstName`, `MiddleName`, `LastName`, `LanguageCodeID`, `MessageTypeCodeID`, and `IPAddress`.
4. **Error Handling**:
    - If the `@ReservationID` is null, an error is raised with the message 'Invalid Reservation' and the procedure returns `-9`.

## Dependencies
- **Tables**:
  - **`ejReservation.dbo.Contact`**: This table stores contact details related to reservations. The procedure reads data from this table using a query.
- **Hints**:
  - **`NOLOCK`**: This hint is used to avoid locking during the data read operation, which can enhance performance but may read uncommitted data.

## Example Usage
The following sample SQL script demonstrates how to call the `[dbo].[Contact_get]` stored procedure:

```sql
DECLARE @ReservationID INT;
SET @ReservationID = 12345;

EXEC [dbo].[Contact_get] @ReservationID;
```

In this example:
- `12345` is the placeholder value for `ReservationID`.
- The stored procedure retrieves and displays contact details for the reservation identifier `12345`.

## Error Handling
The stored procedure employs basic error handling mechanisms:

- If `@ReservationID` is null:
  - The procedure raises an error with the message 'Invalid Reservation'.
  - The procedure returns error code `-9`.
- No TRY-CATCH blocks are used; it relies on the `RAISERROR` function.

## Performance Considerations
- **`NOLOCK` Hint**: This is used to minimize locking contention and improve read performance, though it may return uncommitted data.
- **Validation Check**: Minimal validation (solely checking for non-null `@ReservationID`) helps ensure performance is not significantly impacted by complex checks.

## Version History
- **Initial Version**: Created on 27/07/06 by Edmondv.
- **Current Version**: Contact retrieval procedure version tagged as 1 as of 27/07/06.
  
No additional updates or changes have been recorded for this stored procedure.

By following these guidelines and understanding the provided detailed descriptions, users and developers can effectively utilize the `[dbo].[Contact_get]` stored procedure to retrieve contact details based on a reservation identifier within the `ejReservation` database.