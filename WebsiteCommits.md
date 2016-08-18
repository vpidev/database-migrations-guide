# Connecting database changes to website changes

Database versions are connected to website / web application changes through their commit
 message using the case number that was worked on.

The website commit might look like: 

```
Case 0000 - Add Notes column to my table grid
```

and the accompanying database commit would look like:

```
Case 0000 - Add Notes column to the My_Table_List.sql sproc
```

Then when searching in [Fogbugz](vpidev.fogbugz.com) for the case number it is easy to see
 what version that database was going to be for a given website commit.

***

Some website / web application changes won't need an associated database change, but the needed database version can be determine based on the date the website / web application and database were committed.

This is subject to change / fleshed out as needed.

***

[Back to table of contents](README.md)
