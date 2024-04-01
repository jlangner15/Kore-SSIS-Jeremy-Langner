# Kore SSIS Take Home Assessment 
## Completed by Jeremy Langner on April 1 10am PST
### Directory Structure
**Important: ensure this `initDBKoreScript.sql` script is run first as a new table and 2 new stored procs are necessary for successful results**

`scripts/` folder contains necessary initial script `initDBKoreScript.sql` to run to create the database, create and populate tables, and any stored procedures.


`ssis/kore_ETL_SSIS/` is where the SSIS package solution an be found and extracted to run the package

`users/` contains the `users.csv` file holding users data to perform the ETL process on

`results/` contains `results.xlsx` an excel sheet with prod, stg, and err Users table results after successful completion

`backup/` contains `KoreAssignment_Jeremy_Langner.zip` a SQL database backup

### Assumptions
1. Assume that a 'null' value within a csv record corresponds to a true null value and that the user just didnt insert 'null' as a string input.
In the `Extract User Data` Data Flow object a Derived Column converts 'null' string value to a true DB null value before populating staging database.

2. Assume that a user without a UserID should be removed from staging and placed into a seperate table for further manual review by a developer. Thus I created a another table `err.Users` which gets populated by the `Migrate Invalid User Data to Err Table` Data Flow object via a Lookup for a null UserID value.

3. Assume that a null Age, LastLoginDate, and PurchaseTotal are irrelevant and unimportant for the purpose of the application as users' age may be irrelevant, they may not have logged in or purchased anything.

4. Assume that when a record is going from staging into production to always update that record with staging information that could change
    - LastLoginDate
    - PurchaseTotal
    - RecordLastUpdated

### Design Decisions
- Use `Remove Duplicate Users` which is an *Execute SQL Task* Data Flow Object for data cleaning in part 2. This task calls a stored procedure `stgRemoveDuplicatesNullUsers` to remove duplicates or null names(as seen below). Executing a stored procedure it gives us the flexibility and maintainability to hold all logic and business information within the stored procedure instead of a series of Data Flow operations within the SSIS package for future updates and changes.
    - Duplicate records based on UserID and FullName
    - FullName that start with 'Null%' (which get to moved to err.Users table as from 2. above)    
- The Incremental Load for part 3 is split into two main tasks within the data flow, one to insert from staging and one to update values already in prod. The first task extracts values from staging and performs an existence lookup on UserID in prod and if there is no match then simply insert such record. The second task extracts prod info and looks up UserID in staging and merges with the match in first task lookup on UserID. It then calls a OLE DB Command to execute stored procedure `upsertProdUsers` which takes in staging UserID, LastLoginDate, and PurchaseTotal to update prod Users with given staging values(assumptions from 3. above)

## Thank you for your considerations, I look forward to discussing my learning, implementation, and ideas further with the team!
