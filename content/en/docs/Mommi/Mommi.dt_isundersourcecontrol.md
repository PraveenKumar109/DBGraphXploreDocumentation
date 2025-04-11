# Technical Documentation for Stored Procedure `dbo.dt_isundersourcecontrol`

## Procedure Overview

The stored procedure `dbo.dt_isundersourcecontrol` is designed to verify if a given project is under source control and optionally retrieve a list of objects (procedures) within the project. This procedure addresses a critical business need for ensuring that version control practices are adhered to by enabling checks and listings of controlled objects in a SQL server environment.

## Input Parameters

| Parameter Name  | Data Type   | Purpose                                                                                                                                     | Expected Values / Constraints                                            |
|-----------------|-------------|---------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------|
| `@vchLoginName` | `varchar(255)` | The login name to be used for authentication with the source control system.                                                                 | An optional parameter; can be an empty string.                           |
| `@vchPassword`  | `varchar(255)` | The password to be used for authentication with the source control system.                                                                | An optional parameter; can be an empty string.                           |
| `@iWhoToo`      | `int`       | A flag indicating the operation mode of the procedure. `0` checks if the project is under source control. `1` additionally retrieves a list of objects in the project. | Expected values are `0` or `1`. `0` just checks; `1` retrieves objects.  |

## Output Parameters

This procedure does not use output parameters but may generate result sets or raise errors as necessary.

## Logic Description

1. **Initialization and Variable Declaration:**
   - Several local variables are declared and initialized to store information such as object IDs, project properties, and temporary text.
   
2. **Retrieve Project Properties:**
   - The project properties such as `VCSProject`, `VCSSourceSafeINI`, `VCSSQLServer`, and `VCSSQLDatabase` are retrieved through calls to the `dbo.dt_getpropertiesbyid_vcs` procedure.
   
3. **Validation:**
   - Validates if any of the critical properties are `null` or empty. If true, raises an error `Not Under Source Control`.
   
4. **Mode-specific Operations (`@iWhoToo`):**
   - If `@iWhoToo` is `1`, initiates object listing process:
     - Utilizes OLE Automation (`sp_OACreate`, `sp_OAMethod`, `sp_OAGetProperty`) to interact with the source control system.
     - Retrieves and stores the list of object names in a temporary table `#ObjectList`.
     - Queries and outputs the contents of `#ObjectList`.
   
5. **Error Handling and Cleanup:**
   - Handles errors using labels and GOTO statements.
   - The `E_OAError` label captures OLE Automation errors and calls `dbo.dt_displayoaerror` for error details.
   - The `CleanUp` label performs cleanup before exiting the procedure.

## Dependencies

- **Tables:**
  - **`dbo.dtproperties`**: Used to fetch project-related properties such as `VCSProjectID`.
  
- **Stored Procedures:**
  - **`dbo.dt_getpropertiesbyid_vcs`**: Fetches specific properties by project ID.
  - **`dbo.dt_displayoaerror`**: Handles and displays OLE Automation errors.
  
- **System Stored Procedures:**
  - **`master.dbo.sp_OACreate`**: Creates an OLE object.
  - **`master.dbo.sp_OAMethod`**: Executes a method on an OLE object.
  - **`master.dbo.sp_OAGetProperty`**: Retrieves a property value from an OLE object.

## Example Usage

```sql
-- Example to just check if the project is under source control
EXEC dbo.dt_isundersourcecontrol @vchLoginName = 'admin', @vchPassword = 'password', @iWhoToo = 0;

-- Example to check and get a list of objects under the project
EXEC dbo.dt_isundersourcecontrol @vchLoginName = 'admin', @vchPassword = 'password', @iWhoToo = 1;
```

## Error Handling

- **General Errors:**
  - If the critical project properties are invalid (`null` or empty), the procedure raises a custom error `'Not Under Source Control'`.
  
- **OLE Automation Errors:**
  - Errors during OLE Automation tasks trigger the `E_OAError` label.
  - The procedure `dbo.dt_displayoaerror` is called to display detailed error information.

## Performance Considerations

- **Set NOCOUNT ON:**
  - Using `SET NOCOUNT ON` to minimize the overhead of sending DONE_IN_PROC messages to the client.
  
- **Temporary Tables:**
  - Uses a temporary table `#ObjectList` for storing object names, which should be efficiently managed by ensuring proper clean-up.
  
- **Minimize OLE Automation:**
  - Intensive use of OLE Automation can be resource-heavy. Ensure appropriate resource allocation and error handling.

## Version History (Optional)
- Initial creation and documentation as of [current date].

```yaml
- Version: 1.0
  Date: yyyy-mm-dd
  Description: Initial creation of `dbo.dt_isundersourcecontrol` stored procedure.
```

---

This comprehensive documentation should aid in understanding, utilizing, and maintaining the `dbo.dt_isundersourcecontrol` stored procedure effectively.