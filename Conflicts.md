# Concurrent Development

## Dealing with conflicts

When setting up migration scripts, you may find that another developer has made changes since you last checked out migration repository. Let's use a simple example to demonstrate:

Developer A is making a table change, creating a script in the `up` folder named

    0003 - Alter SKU_Code column in Product_SKU table.sql

Developer B, at the same time has committed a migration `up` file containing the file

    0003 - Add column Size to Parts table.sql

At this point, there are effectively two branches, in most cases this is acceptable (as long as the changes don't actually impact each other) the migration scripts are simply run in alphabetical order, so when the two branches are *merged*, the up folder will look like this:

    up\
        0001 - <first merge script example>.sql
        0002 - <merge script example>.sql
        0003 - Add column Size to Parts table.sql
        0003 - Alter SKU_Code column in Product_SKU table.sql

and will be run in the in order shown (Developer B's changes will be run before Developer A's)

If the two different migrations conflict with each other (i.e. One migration script deletes the table that the next would update) Then the erroring migration script needs to be renamed by updating the prefix number to place it in the desired order.

<!--
## Guarding

[I think guarding shouldn't be a recommended practice, but if the developer knows how and when to guard they are ok to do so]

Guarding a script is to make it safe to run multiple times, and doing so will reduce the chance of failure for your script during a conflict.
Guarding is a way to write something so that it checks if what it is going to do is safe to do.

A hypothetical example would be if two developers both had the same version of the database and in both of their branches they decided to add a column to the same table.  When they would merge it would cause the second developer's script to fail; however, if they write their scripts in a guarded way the first script will execute successfully and the second script won't effectively be skipped over.

### Case 1: Altering a table

An un-guarded script would look like this:

```sql
ALTER TABLE dbo.My_Table
ADD Notes VARCHAR(50)
```

This will fail if there is already a column named `Notes` in the `dbo.My_Table` table.

To write this in *guarded* way would look like this:

```sql
IF NOT EXISTS(
    SELECT *
    FROM sys.columns c
    JOIN sys.tables t
    ON c.object_id = t.object_id
    JOIN sys.schemas s
    ON t.schema_id = s.schema_id
    WHERE s.name = 'dbo'
    AND t.name = 'My_Table'
    AND c.name = 'Notes'
)
BEGIN
    ALTER TABLE dbo.My_Table
    ADD Notes VARCHAR(50)
END
```

### Case 2: Inserting data

An unguarded insert script would look like this:

```sql
INSERT INTO dbo.My_Table
(Name, Code, Notes)
VALUES
('Name1', 'Code1', 'Notes1')
```

This will insert that record every time the script is run (possibly not ideal).

To guard this script and prevent duplicate entries it would look like this:

```sql
IF NOT EXISTS(
    SELECT *
    FROM dbo.My_Table
    WHERE Name = 'Name1'
    AND Code = 'Code1'
    AND Notes = 'Notes1'
)
BEGIN
    INSERT INTO dbo.My_Table
    (Name, Code, Notes)
    VALUES
    ('Name1', 'Code1', 'Notes1')
END
```
-->
***

[Back to table of contents](README.md)
