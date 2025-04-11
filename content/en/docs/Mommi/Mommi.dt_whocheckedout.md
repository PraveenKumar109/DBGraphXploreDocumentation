# Technical Document for Stored Procedure `dbo.dt_whocheckedout`

## Procedure Overview

The `dbo.dt_whocheckedout` stored procedure is designed to identify and retrieve the user who has checked out a specified SQL object, such as a stored procedure, from SQLVersionControl. This can be particularly useful in environments where multiple developers or administrators are working on database objects, and you need to ascertain who is currently working on a given object. The procedure leverages Version Control System (VCS) properties to facilitate this tracking.

## Input Parameters

- **@chObjectType** (char(4)): 
  - **Purpose:** Specifies the type of SQL object for which the procedure is fetching checkout information.
  - **Expected Values:** Fixed length string representing the SQL object type. For this procedure, it expects the value 'PROC' for stored procedures.
  - **Constraints:** Must be exactly 4 characters long.

- **@vchObjectName** (varchar(255)): 
  - **Purpose:** The name of the SQL object for which the procedure is fetching checkout information.
  - **Expected Values:** Up to 255 characters long representing a valid SQL object name.
  - **Constraints:** Must be a valid and existing object name.

- **@vchLoginName** (varchar(255)): 
  - **Purpose:** The login name for authentication to the SQL Server where the version control system is managed.
  - **Expected Values:** Up to 255 characters long representing a valid SQL login.
  - **Constraints:** Must be a valid SQL Server login name.

- **@vchPassword** (varchar(255)): 
  - **Purpose:** The password corresponding to the provided login name.
  - **Expected Values:** Up to 255 characters long.
  - **Constraints:** Must correspond to the provided login name.

## Output Parameters

There are no designated output parameters; however, the procedure returns a string indicating the user who has checked out the specified object.

## Logic Description

1. **Initialization and Variable Declaration:**
   - Initializes various variables to store object IDs, return values, and property values.
   
2. **Setting Up VCS Properties:**
   - Fetches the Version Control System properties such as the project name, SourceSafe INI path, SQL Server name, and database name from a properties table `dtproperties`.

3. **Object Type Check:**
   - Checks if the object type (`@chObjectType`) is 'PROC'. If true, proceeds with the operation; otherwise, the procedure does nothing.

4. **Creating OLE Automation Object:**
   - Creates an OLE automation object using `sp_OACreate`. If this fails, it jumps to the error handling label `E_OAError`.

5. **Calling `WhoCheckedOut` Method:**
   - Calls the `WhoCheckedOut` method via `sp_OAMethod` on the created automation object to fetch the user who checked out the SQL object. Passes the project and object details fetched earlier.

6. **Error Handling:**
   - If any errors occur during the OLE operation, jumps to `E_OAError` label where it calls another stored procedure `dt_displayoaerror` to display the error message.
   
7. **Return Value:**
   - Finally, the checked-out user (if found) is selected and returned.

8. **Clean-Up:**
   - Releases resources and exits the procedure.

## Dependencies

- **Tables:**
  - **dtproperties:** Stores various properties including the VCS Project ID used for fetching other VCS-related details.
  
- **Procedures:**
  - **dt_getpropertiesbyid_vcs:** Fetches VCS properties by property ID.
  - **dt_displayoaerror:** Displays OLE automation errors.

## Example Usage

```sql
DECLARE @checkedOutUser VARCHAR(255);

EXEC dbo.dt_whocheckedout 
    @chObjectType = 'PROC', 
    @vchObjectName = 'usp_GetUserDetails', 
    @vchLoginName = 'admin', 
    @vchPassword = 'password123';

-- The output will be the name of the user who has checked out the procedure `usp_GetUserDetails`
```

## Error Handling

- **TRY-CATCH Blocks:** This procedure does not explicitly use TRY-CATCH blocks but handles errors using GOTO statements.
- **Specific Error Handling:**
  - **OLE Automation Errors:** Uses `sp_OACreate` and `sp_OAMethod` return values to detect errors, and jumps to `E_OAError` label to handle them.
  - Calls `dt_displayoaerror` to log and display errors that occur during OLE operations.

## Performance Considerations

- **Setting NOCOUNT ON:** Reduces network traffic by preventing the sending of DONE_IN_PROC messages to the client for each statement.
- **Minimal Access to Tables:** The procedure only accesses the `dtproperties` table to fetch required property values, minimizing the load on the database.
- **Error Management:** Efficient error handling ensures that resources are freed and transactions are not left open.

## Version History

- **v1.0.0:** Initial version of the stored procedure.

This document provides a comprehensive overview and technical details necessary for understanding and utilizing the `dbo.dt_whocheckedout` stored procedure effectively.