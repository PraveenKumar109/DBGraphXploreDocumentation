---
title: CustomerPaymentsCardDataRemediation
weight: 20
---
## Procedure Overview

### Purpose
The `dbo.CustomerPaymentsCardDataRemediation` stored procedure is designed to remediate sensitive credit card data from the `CustomerPayments` database. This procedure updates or masks sensitive encrypted and tokenized payment data to ensure compliance with PCI (Payment Card Industry) data security standards.

### Business Use Case
This procedure is part of a routine data remediation process that aims to protect and secure credit card information used in customer payments. It is called daily by the `PCI_DataRemediationJob` to ensure that data exposure is minimized and outdated sensitive information is appropriately handled.

## Input Parameters

1. **@TimeBoundaryForDataRemediation** (`INTEGER`, Default: NULL)
   - Purpose: Defines the age of data (in hours) eligible for remediation.
   - Expected Values: Any integer value. When NULL, defaults to 24 hours.

2. **@TimeWindowToPickSecureData** (`INTEGER`, Default: NULL)
   - Purpose: Specifies the age of secure data (in weeks) to be considered for remediation.
   - Expected Values: Any integer value. When NULL, defaults to 2 weeks.

3. **@PerformanceThreshold** (`INTEGER`, Default: NULL)
   - Purpose: Sets the performance threshold for the procedure's execution time (in seconds).
   - Expected Values: Any integer value. When NULL, defaults to 60 seconds.

## Output Parameters

This stored procedure does not utilize output parameters.

## Logic Description

### Step-by-Step Explanation

1. **Initialize Variables and Context**
   - Declares and initializes various variables for tracking job execution, error handling, date-time, and remediation counts.

2. **Set Context Information**
   - Sets the `CONTEXT_INFO` for maintaining the execution context.

3. **Calculate Default Values**
   - Assigns default values to input parameters if they are NULL.

4. **Create Temporary Tables**
   - Creates temporary tables `#SecureDataNonRemediated`, `#TotalNonRemediatedTransactions`, and `#TransactionsToBeRemediated` to store intermediate results of the remediation process.

5. **Populate Temporary Tables**
   - Inserts records into `#SecureDataNonRemediated` for secure data without corresponding saved payment methods created within a specified time window.
   - Further inserts records into `#TotalNonRemediatedTransactions` to identify transactions with non-remediated data.
   - Counts transactions that are not eligible for remediation based on specific criteria.

6. **Determine Transactions for Remediation**
   - Identifies transactions that need remediation based on their encryption method and status and inserts them into `#TransactionsToBeRemediated`.

7. **Batch Processing of Remediation**
   - Performs remediation in batches of records, updating encrypted data and tokenized data accordingly using `REPLICATE` to mask the sensitive information.
   - Uses `TRY-CATCH` block to handle any errors during batch processing.

8. **Update Job Tracker Table**
   - Inserts job execution details into `PCIRemediationJobTracker` to log the job's completion status and performance metrics.

9. **Send Notification**
   - Executes a notification stored procedure to inform about the job's status.

## Dependencies

### Tables
- `ejCustomerPayments..SecureData`
- `ejCustomerPayments..SavedPaymentMethod`
- `ejCustomerPayments..Payment`
- `ejCustomerPayments.dbo.PCIRemediationJobTracker`

### Stored Procedures
- `dbo.CardDataRemediationJob_Notification`

## Example Usage

```sql
EXEC dbo.CustomerPaymentsCardDataRemediation 
     @TimeBoundaryForDataRemediation = 24, 
     @TimeWindowToPickSecureData = 2, 
     @PerformanceThreshold = 60;
```
### Sample Input
- `@TimeBoundaryForDataRemediation` : 24 (hours)
- `@TimeWindowToPickSecureData` : 2 (weeks)
- `@PerformanceThreshold` : 60 (seconds)

### Expected Results
This would start the remedial process for data older than 24 hours benchmarked over the last 2 weeks, while maintaining an execution performance threshold of 60 seconds. The completion status and performance metrics will be recorded in the `PCIRemediationJobTracker` table.

## Error Handling

The procedure employs a `TRY-CATCH` block around the core logic for batch updating sensitive data. If an error occurs, relevant error details are captured using `ERROR_NUMBER()` and `ERROR_MESSAGE()`, and the job's execution is logged with a failure status (`'R'`). The batch operation is stopped upon encountering an error.

## Performance Considerations

### Optimizations
- **Batch Processing**: Handles updates in batches (`@BatchSize = 5000`) to manage large data sets efficiently and avoid locking issues.
- **Indexes**: Uses clustered indexes on temporary tables to optimize join operations and query performance.
- **NOLOCK and Specific Indexes**: Employed `NOLOCK` hints and specific indexes for read operations to minimize locking and improve read performance.

### Potential Bottlenecks
- High volume of data to process may occasionally breach the performance threshold.
- Extensive use of temporary tables and batch processing may require sufficient tempdb resources.

### Recommendations
- Adjust batch size (`@BatchSize`) based on the server capacity and workload.
- Regularly monitor the `PCIRemediationJobTracker` for performance trends and adjust parameters accordingly.
- Ensure that the `ejCustomerPayments` database and all related tables are properly indexed and maintained.

## Version History

| Changed On  | Changed By | Defect Ref   | Change Description                                   |
|-------------|------------|--------------|------------------------------------------------------|
| 03 Apr 2017 | SanjayH    | PAYNEW-5026  | Excluded remediation of bank transfer payments       |
| 24 Oct 2017 | SanjayH    | PAYNEW-6930  | Include bank details for ELV payments in regular job |
| 01 Nov 2017 | SanjayH    | PAYNEW-6949  | Do not remediate EncryptedData if Tokens are used    |
| 12 Dec 2017 | SanjayH    | PAYNEW-7557  | Non-Remediated trans count issue due to Tokens       |
| 25 Jan 2018 | SanjayH    | PAYNEW-7557  | Remediation tokens                                   |
| 31 Jan 2018 | SanjayH    | PAYNEW-7689  | Changes post performance enhancements                |

This ensures that the stored procedure stays robust and efficient in handling sensitive credit card data remediation.