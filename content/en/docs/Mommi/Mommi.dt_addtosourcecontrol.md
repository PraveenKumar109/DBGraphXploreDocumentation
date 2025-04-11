# Technical Document for Stored Procedure `dbo.dt_addtosourcecontrol`

## Procedure Overview
The `dbo.dt_addtosourcecontrol` stored procedure is designed to integrate a database project with a source control system, specifically Visual SourceSafe (VSS). Its primary purpose is to manage version control for database objects by creating projects in VSS and setting various associated properties. This procedure is essential for maintaining the integrity and version history of database projects, thus addressing the business use case of database version control.

## Input Parameters
| Parameter Name        | Data Type       | Purpose                                                                      | Expected Values / Constraints                         |
|-----------------------|-----------------|-------------------------------------------------------------------------------|-------------------------------------------------------|
| `@vchSourceSafeINI`   | `varchar(255)`  | Path to the SourceSafe INI file.                                              | A valid file path to a SourceSafe INI configuration.   |
| `@vchProjectName`     | `varchar(255)`  | Name of the project to be created in SourceSafe.                              | A unique and descriptive project name.                |
| `@vchComment`         | `varchar(255)`  | Comments or description for the project when adding it to SourceSafe.         | Any relevant description or comments.                 |
| `@vchLoginName`       | `varchar(255)`  | Login name for accessing SourceSafe.                                          | A valid SourceSafe username.                          |
| `@vchPassword`        | `varchar(255)`  | Password for the SourceSafe login name.                                       | A valid SourceSafe password.                          |

## Output Parameters
This stored procedure does not have explicit output parameters. However, it does return two values via `SELECT` statements:
1. `@vchProjectName` - The name of the project created in SourceSafe.
2. `@iObjectCount` - An integer indicating the object count, although always set to `0` in this procedure.

## Logic Description
1. **Initialization and Declarations:**
   - Several variables are initialized to hold return values, object IDs, and configuration settings.

2. **Create VSS Object:**
   - The procedure attempts to create an instance of the VSS automation object using `sp_OACreate`. If unsuccessful, it jumps to the `E_OAError` error handling label.

3. **Add Project to SourceSafe:**
   - Executes the `AddProjectToSourceSafe` method of the VSS object to create a new project in SourceSafe with the specified parameters (`@vchSourceSafeINI`, `@vchProjectName`, @@SERVERNAME, `@vchDatabaseName`, `@vchLoginName`, `@vchPassword`, and `@vchComment`). If unsuccessful, it jumps to the `E_OAError` error handling label.

4. **Set Database Properties:**
   - Begins a transaction named `SetProperties`.
   - Executes `dbo.dt_adduserobject_vcs` to add a high-level user-defined object and retrieves the object ID as `@iPropertyObjectId`.
   - Converts `@iPropertyObjectId` to a string and assigns to `@vchParentId`.
   - Sets various properties (`VCSProjectID`, `VCSProject`, `VCSSourceSafeINI`, `VCSSQLServer`, `VCSSQLDatabase`) for the VSS project using `dbo.dt_setpropertybyid`.
   - If an error occurs during property setting, it jumps to the `E_General_Error` error handling label; otherwise, commits the transaction.

5. **Cleanup and Return:**
   - Sets `@iObjectCount` to `0`.
   - Navigates to the `CleanUp` label, which returns the `@vchProjectName` and `@iObjectCount`.

6. **Error Handling:**
   - `E_General_Error`: A general error handling label that navigates to the `CleanUp` label.
   - `E_OAError`: A specific error handler that displays OLE Automation errors using `dbo.dt_displayoaerror` and then navigates to the `CleanUp` label.

## Dependencies
### Tables and Views
None explicitly referenced within the procedure.

### Stored Procedures
- `master.dbo.sp_OACreate`: Creates an OLE Automation object.
- `master.dbo.sp_OAMethod`: Calls a method on the created OLE Automation object.
- `dbo.dt_adduserobject_vcs`: Adds a high-level user-defined object for version control (Details not provided).
- `dbo.dt_setpropertybyid`: Sets properties for the specified object (Details not provided).
- `dbo.dt_displayoaerror`: Displays OLE Automation errors (Details not provided).

## Example Usage
```sql
DECLARE @projectName VARCHAR(255);
EXEC dbo.dt_addtosourcecontrol
    @vchSourceSafeINI = 'C:\VSS\srcsafe.ini',
    @vchProjectName = @projectName OUTPUT,
    @vchComment = 'Initial project setup',
    @vchLoginName = 'admin',
    @vchPassword = 'password';
    
SELECT @projectName AS ProjectName;
```
Expected output: `ProjectName` will contain the name of the project created in SourceSafe.

## Error Handling
The procedure employs two primary error handling mechanisms:
1. **General Error Handling (`E_General_Error`):** This is invoked when any general error is detected during property setting. It ensures that the procedure flows to cleanup without specifying detailed error messages.
2. **OLE Automation Error Handling (`E_OAError`):** This handles errors specific to OLE Automation operations by calling `dbo.dt_displayoaerror` to display the error and then navigating to cleanup.

## Performance Considerations
- The procedure uses transactions to ensure atomic property setting operations.
- No explicit performance optimizations (e.g., indexing, query optimization) are noted within the procedure.
- Potential performance issues might stem from the use of OLE Automation stored procedures, which can be resource-intensive.

## Version History
Version history is not provided in the current definition, but it is recommended to maintain this section for tracking updates and changes.

---

This document provides a comprehensive overview of the stored procedure `dbo.dt_addtosourcecontrol`, detailing its parameters, logic, dependencies, usage, error handling, and performance considerations for effective use in database version control integration with VSS.