> [!NOTE] Delta Load
> To load those data from the OLTP that changed since the last time we pulled the data.

There are various approaches
1. Write Ahead Log [[Log-based]]
	1. Benefits:
		1. Most efficient, as there is minimal stress on the OLTP database when loading the data. Since the database already writes the WAL for durability, CDC can just read that log
		2. Don't need to figure out the changes as WAL already does it
	2. Cons:
		1. Complex to implement
		2. Doesn't work for initial load of data, since the logs only hold recent changes (First time extracting data from somewhere)
2. Additional Columns [[Watermark]]
	1. To have `created_at` & `updated_at` columns to make it easier to query the table to find out what are the latest changes. These columns have to be created on the OLTP side and the warehouse needs to track the watermark to figure out to extract from which row / date onwards
	2. Benefits:
		1. Full load and Delta Load is the same approach, don't need to invent something new
	3. Cons:
		1. Application side need to implement these column to each table
		2. Each worker that performs (`INSERT, UPDATE) has to update these columns
		3. Unable to track `DELETE`
3. Database Triggers [[Trigger-based]]
	1. Whenever the database changes, to insert these changes to a dedicated table on `FOR EACH ROW` there is a trigger for `INSERT, UPDATE & DELETE`
	2. Benefits:
		1. Able to catch all `INSERT, UPDATE & DELETE` and write them into a log table
	3. Cons:
		1. Increase load on the OLTP database as they will need to implement an additional trigger on the above actions. As the trigger runs in the same `transaction`, which lengthens the transactions making the locks longer
		2. Log table can get very huge
4. [[Scan & Hash Table]]
	1. Scan the rows for the table, generate a hash for each row and compare if the data already exist in the warehouse
	2. Benefits:
		1. Simple to implement
	3. Cons:
		1. Does not scale as can be very computationally expensive on large tables


