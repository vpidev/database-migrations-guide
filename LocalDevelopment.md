# Local Development

## Restore development database from prepared backup

There are multiple ways to restore your database from a backup.  This is the preferred way:

1. Provided in the repository folder is a file called `LOCALDB.DBDeployment-Restore.bat`. 
    - This file uses the `--restore` and `--restorefrompath` parameters in the call to RoundhousE which will restore the database before running migrations.
2. Edit the `LOCALDB.DBDeployment-Restore.bat` to point to the correct location of the `*.bak` backup file.
3. Run the `LOCALDB.DBDeployment-Restore.bat` file.

## Run local migrations using supplied script

Running migrations locally should normally be done using the supplied `LOCAL.DBDeployment.bat` file found with the repo.
This will run RoundhousE in the preferred way for local development.

## Local development process example

1. As a developer you've been given the task of adding a column to a search page in an application. You know that you'll need to do database changes for this column to appear.  The changes include:
    - Update the table to have the new column.
        - The table is called `dbo.My_Table` and it already exists on the `ENTERPRISE` database.
            - It looks like this:

                Column | Definition
                --- | ---
                My_Table_PK | INT IDENTITY(1,1) PRIMARY KEY
                Name | VARCHAR(50)
                Code | VARCHAR(50)

        - The column will be named `Family_FK`
        - The column must have referential integrity to the `dbo.Family` table on the `Family_PK` column.
    - Add some default data to the column.
        - All existing records need to have a value of `1`.
        - The table `dbo.Family` already has a record with the `Family_PK` of `1`.
    - Change the 4 of the 5 stored procedures to interact with the new column. The stored procedures are:
        - `dbo.My_Table_Insert` - The definition is located in `~ENTERPRISE\sprocs\dbo.My_Table_Insert.sql`
        - `dbo.My_Table_Update` - The definition is located in `~ENTERPRISE\sprocs\dbo.My_Table_Update.sql`
        - `dbo.My_Table_Get_By_Key` - The definition is located in `~ENTERPRISE\sprocs\dbo.My_Table_Get_By_Key.sql`
        - `dbo.My_Table_List` - The definition is located in `~ENTERPRISE\sprocs\dbo.My_Table_List.sql`
        - `dbo.My_Table_Delete` - This doesn't need to change to work with the new column.
2. The first thing you should do is check if there have been updates to the `ENTERPRISE` repository before beginning, and pull them down.
    - If there are changes then run RoundhousE to bring your database into sync with `dev`.
3. To do the database changes first you'll need to add the new column to the table.  To do so:
    1. Open the `ENTERPRISE` folder and navigate to the `up` folder.
    2. Decide what is the next available sequence number.  For this is example it will be `0110`.
    3. Create a file named `0110 - Add Family_FK to the dbo.My_Table table.sql`
    4. Open the file in SSMS to add this code:
        ```sql
        ALTER TABLE dbo.My_Table
        ADD Family_FK int CONSTRAINT FK_dbo_My_Table_Family FOREIGN KEY(Family_FK) REFERENCES dbo.Family(Family_PK)
        GO
        UPDATE dbo.My_Table
        SET Family_FK = 1
        ```
    5. Open the `sprocs` folder and find the `dbo.My_Table_Insert.sql` and open in SSMS
        - It currently looks like this:
            ```sql
            ALTER PROCEDURE dbo.My_Table_Insert
                @Name VARCHAR(50),
                @Code VARCHAR(50)
            AS
            BEGIN
            SET NOCOUNT ON;
                INSERT INTO dbo.My_Table
                (Name, Code)
                VALUES
                (@Name, @Code)
            END
            ```
        - We will edit it to have a Family_FK option:
            ```sql
            ALTER PROCEDURE dbo.My_Table_Insert
                @Name VARCHAR(50),
                @Code VARCHAR(50),
                @Family_FK int
            AS
            BEGIN
            SET NOCOUNT ON;
                INSERT INTO dbo.My_Table
                (Name, Code, Family_FK)
                VALUES
                (@Name, @Code, @Family_FK)
            END
            ```
    6. Find the `dbo.My_Table_Update.sql` and open in SSMS
        - It currently looks like this:
            ```sql
            ALTER PROCEDURE dbo.My_Table_Update
                @My_Table_PK int,
                @Name VARCHAR(50),
                @Code VARCHAR(50)
            AS
            BEGIN
                UPDATE dbo.My_Table
                SET Name = @Name,
                    Code = @Code
                WHERE My_Table_PK = @My_Table_PK
            END
        - We will edit it to have a Family_FK option:
            ```sql
            ALTER PROCEDURE dbo.My_Table_Update
                @My_Table_PK int,
                @Name VARCHAR(50),
                @Code VARCHAR(50),
                @Family_FK int
            AS
            BEGIN
                UPDATE dbo.My_Table
                SET Name = @Name,
                    Code = @Code,
                    Family_FK = @Family_FK
                WHERE My_Table_PK = @My_Table_PK
            END
            ```
    7. Find the `dbo.My_Table_Get_By_Key.sql` and open in SSMS
        - It currently looks like this:
            ```sql
            ALTER PROCEDURE dbo.My_Table_Get_By_Key
                @My_Table_PK int
            AS
            BEGIN
                SELECT My_Table_PK, Name, Code
                FROM dbo.My_Table
                WHERE My_Table_PK = @My_Table_PK
            END
            ```
        - We will edit it to have a Family_FK option:
            ```sql
            ALTER PROCEDURE dbo.My_Table_Get_By_Key
                @My_Table_PK int
            AS
            BEGIN
                SELECT My_Table_PK, Name, Code, Family_FK
                FROM dbo.My_Table
                WHERE My_Table_PK = @My_Table_PK
            END
            ```
    8. Find the `dbo.My_Table_List.sql` and open in SSMS
        - It currently looks like this:
            ```sql
            ALTER PROCEDURE dbo.My_Table_List
            AS
            BEGIN
                SELECT My_Table_PK, Name, Code
                FROM dbo.My_Table
            END
            ```
        - We will edit it to have a Family_FK option:
            ```sql
            ALTER PROCEDURE dbo.My_Table_List
            AS
            BEGIN
                SELECT My_Table_PK, Name, Code, Family_FK
                FROM dbo.My_Table
            END
            ```
    9. Our feature is now implemented on the database we can run RoundhousE via the provided `.bat` file.
        - Our table now looks like this:

                Column | Definition
                --- | ---
                My_Table_PK | INT IDENTITY(1,1) PRIMARY KEY
                Name | VARCHAR(50)
                Code | VARCHAR(50)
                Family_FK | INT
        
        - And the sprocs that interact with it are updated

    10. We now need to commit our changes to the `ENTERPRISE` repository
        - Pull once more, this will give you an opportunity to merge your changes with the latest migration before commiting your work.
        - We can use a commit message like:
            ```
            Case 0000 - Add Family_FK to dbo.My_Table and related sprocs
            ```
        - We would then commit the change and push to Kiln so that others can access them.
***

[Back to table of contents](README.md)
