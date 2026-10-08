# tn-skill-project
import data using transform maps (spreadsheet)
Import Data Using Transform Maps (Spreadsheet)
1. Project Overview
Objective
The objective of this project is to import data from an Excel spreadsheet into a ServiceNow table using Import Sets and Transform Maps.

The implementation demonstrates how to:

Prepare spreadsheet data.
Upload spreadsheet data into ServiceNow.
Create an Import Set.
Create a Transform Map.
Map source fields to target fields.
Configure Coalesce.
Transform and validate records.
Verify the imported data.
Data Flow
Excel Spreadsheet
       ↓
   Import Set
       ↓
Import Set Table
       ↓
  Transform Map
       ↓
Field Mapping / Validation
       ↓
 Target ServiceNow Table
2. Prerequisites
The following are required:

ServiceNow Developer/Personal Developer Instance.
Appropriate administrative or import permissions.
Excel spreadsheet containing the source data.
A target ServiceNow table.
Basic knowledge of ServiceNow tables and fields.
For this project, create an Excel file named:

employee_import.xlsx
Use the following sample data:

Employee Number	First Name	Last Name	Email	Department	Location
1001	John	Smith	john.smith@example.com	IT	Chennai
1002	Mary	Johnson	mary.johnson@example.com	HR	Bangalore
1003	David	Brown	david.brown@example.com	Finance	Mumbai
1004	Sarah	Wilson	sarah.wilson@example.com	IT	Hyderabad
1005	Robert	Davis	robert.davis@example.com	HR	Pune
The first row should contain the column headers.

3. Create and Upload the Import Set
Navigate to:

All → System Import Sets → Load Data
Upload:

employee_import.xlsx
Select the appropriate file type and start the upload.

ServiceNow loads the spreadsheet into an Import Set staging table.

The staging table may contain fields similar to:

u_employee_number
u_first_name
u_last_name
u_email
u_department
u_location
Review the imported data before continuing.

Verify that:

All expected columns are available.
The number of rows is correct.
No unexpected blank values exist.
Column headers are correct.
Employee numbers are unique.
The Import Set acts as temporary storage for the external data before it is transformed into the target table.

4. Create the Transform Map
Navigate to:

All → System Import Sets → Create Transform Map
Create a new Transform Map.

Example configuration:

Transform Map Name:
Employee Spreadsheet Transform

Source Table:
u_employee_import

Target Table:
u_employee
The target table can be an existing ServiceNow table or a custom table created for the project.

The Transform Map controls how data moves from the Import Set table to the target table.

4.1 Field Mapping
Create the following mappings:

Source Field	Target Field
uemployeenumber	uemployeenumber
ufirstname	ufirstname
ulastname	ulastname
u_email	u_email
u_department	u_department
u_location	u_location
Make sure every required target field is mapped correctly.

If the source and target field names are different, create the mapping manually.

For example:

Source:
u_employee_number

Target:
u_employee_id
5. Configure Coalesce
Coalesce determines whether ServiceNow should insert a new record or update an existing record.

For this project, use:

Employee Number
as the Coalesce field.

Configure:

Source Field:
u_employee_number

Target Field:
u_employee_number

Coalesce:
true
The process works as follows:

Employee Number
       ↓
Does record exist?
    /       \
  Yes       No
   ↓         ↓
Update     Insert
For example, if employee 1001 already exists and the spreadsheet contains employee 1001 again, ServiceNow updates the existing record rather than creating a duplicate.

This is one of the most important parts of the Transform Map configuration.

6. Transform Script for Validation
Transform Scripts can be used when additional processing or validation is required.

An example onBefore Transform Script is:

(function runTransformScript(source, map, log, target) {

    // Validate Employee Number
    if (!source.u_employee_number) {
        ignore = true;
        log.info('Record ignored: Employee Number is empty.');
        return;
    }

    // Validate Email
    if (!source.u_email) {
        ignore = true;
        log.info('Record ignored: Email is empty.');
        return;
    }

    // Remove unnecessary spaces
    if (source.u_first_name) {
        target.u_first_name =
            source.u_first_name.toString().trim();
    }

    if (source.u_last_name) {
        target.u_last_name =
            source.u_last_name.toString().trim();
    }

    // Normalize email
    if (source.u_email) {
        target.u_email =
            source.u_email.toString().trim().toLowerCase();
    }

})(source, map, log, target);
This script:

Checks whether Employee Number exists.
Checks whether Email exists.
Ignores invalid records.
Removes unnecessary spaces.
Converts email addresses to lowercase.
7. Run the Transform
After completing the Transform Map configuration, run the transformation.

The complete process is:

Excel File
    ↓
Import Set
    ↓
Import Set Table
    ↓
Transform Map
    ↓
Field Mapping
    ↓
Coalesce / Validation
    ↓
Target Table
ServiceNow processes each imported row.

A record can result in:

INSERT
UPDATE
IGNORE
ERROR
depending on the configuration and source data.

8. Validate the Transformation
After the transformation completes, review the Transform History.

Check:

Total Records
Inserted
Updated
Ignored
Errors
For example:

Total:     5
Inserted:  5
Updated:   0
Ignored:   0
Errors:    0
A successful import should have no unexpected errors.

Next, open the target table and verify the records.

Example:

Employee Number	Name	Email	Department
1001	John Smith	john.smith@example.com	IT
1002	Mary Johnson	mary.johnson@example.com	HR
1003	David Brown	david.brown@example.com	Finance
1004	Sarah Wilson	sarah.wilson@example.com	IT
1005	Robert Davis	robert.davis@example.com	HR
9. Testing
Testing should be performed to verify both insert and update functionality.

Test Case 1 — New Record
Add the following to the spreadsheet:

1006 | James | Taylor | james.taylor@example.com | IT | Chennai
Expected result:

New record is created.
Test Case 2 — Existing Record
Modify employee 1001:

1001 | John | Smith | john.updated@example.com | IT | Chennai
Run the import again.

Expected result:

Existing employee 1001 is updated.
No duplicate record is created.
Test Case 3 — Missing Email
Add:

1007 | Emily | Green | | HR | Chennai
Expected result:

Record is ignored or rejected
because Email is missing.
Test Case 4 — Duplicate Employee Number
Add two records with the same Employee Number.

Example:

1008 | Alex | Brown | alex@example.com | IT | Chennai
1008 | Michael | Brown | michael@example.com | IT | Chennai
Verify the behavior according to the configured Coalesce logic.

10. Troubleshooting
Incorrect Field Mapping
Problem: Data appears in the wrong target field.

Solution: Review the Transform Map field mappings and make sure each source field is mapped to the correct target field.

Duplicate Records
Problem: Multiple records are created for the same employee.

Solution: Configure a unique field such as Employee Number as the Coalesce field.

Record Not Created
Possible causes:

Required field is missing
Transform Script uses ignore = true
Invalid reference value
Business rule prevents insertion
Incorrect Transform Map
Review the Transform History and system logs.

Reference Field Error
If Department or Location is configured as a reference field, the value in the spreadsheet must correspond to an existing reference record.

For example:

Spreadsheet:
IT

Target:
Reference to IT Department
If the referenced record does not exist, the transformation may fail.

11. Best Practices
Follow these practices when using Transform Maps:

Use meaningful names for Import Sets and Transform Maps.
Validate spreadsheet data before uploading.
Use a unique field for Coalesce.
Test imports in a development instance.
Review Transform History after every test.
Avoid duplicate spreadsheet headers.
Keep a backup of the original spreadsheet.
Validate required fields before transformation.
Test both new and existing records.
Verify the final records in the target table.
12. Completion Checklist
The project is complete when the following items are successfully completed:

[✓] Excel spreadsheet prepared
[✓] Spreadsheet uploaded
[✓] Import Set created
[✓] Import Set table contains data
[✓] Transform Map created
[✓] Correct target table selected
[✓] Source fields mapped to target fields
[✓] Employee Number configured as Coalesce
[✓] Validation script configured
[✓] Transform executed
[✓] New records inserted
[✓] Existing records updated
[✓] Duplicate records prevented
[✓] Invalid records handled
[✓] Transform History reviewed
[✓] Target records verified
[✓] No unexpected errors
13. Final Result
The completed implementation provides a reliable way to import spreadsheet data into ServiceNow.

The final architecture is:

             Excel Spreadsheet
                     |
                     ↓
              Import Set
                     |
                     ↓
             Staging Table
                     |
                     ↓
              Transform Map
                     |
          +----------+----------+
          |                     |
          ↓                     ↓
   Field Mapping          Transform Script
          |                     |
          +----------+----------+
                     |
                     ↓
              Coalesce Check
                     |
             +-------+-------+
             |               |
           Exists          New
             |               |
             ↓               ↓
           UPDATE          INSERT
             \               /
              \             /
               ↓           ↓
             Target ServiceNow
                  Table
