# COFORGE

MySQL and Python can be used together to perform a variety of data-related tasks. Here are some common use cases for collaborating between MySQL and Python:

1. Connecting to MySQL using Python: Python has several libraries that can be used to connect to a MySQL database, including mysql-connector-python, pymysql, and pyodbc.
2. Querying data from MySQL using Python: Once you've connected to your MySQL database using Python, you can execute queries to retrieve data.
3. Modifying data in MySQL using Python: You can use Python to modify data in your MySQL database by executing SQL queries.
4. Loading data from Python to MySQL: You can use Python to load data from external sources (e.g. CSV files) into MySQL.
5. Overall, the combination of MySQL and Python provides a powerful set of tools for managing data.


# Types of SQL commands

In SQL, there are several types of commands that can be used to interact with a database. Here are some commonly used types of SQL commands:
1.	Data Manipulation Language (DML) commands:
o	SELECT: Retrieves data from one or more database tables.
o	INSERT: Inserts new rows of data into a table.
o	UPDATE: Modifies existing data in a table.
o	DELETE: Removes rows of data from a table.



2.	Data Definition Language (DDL) commands:

o	CREATE: Creates a new database, table, view, index, or other database objects.
o	ALTER: Modifies the structure of an existing database object, such as a table.
o	DROP: Deletes an existing database object, such as a table or view.
o	TRUNCATE: Removes all data from a table, but keeps the table structure intact.

3.	Data Control Language (DCL) commands:
o	GRANT: Provides user access privileges to database objects.
o	REVOKE: Revokes user access privileges from database objects.

4.	Transaction Control commands:
o	COMMIT: Saves all changes made within a transaction.
o	ROLLBACK: Undoes all changes made within a transaction.
o	SAVEPOINT: Sets a point within a transaction to which you can roll back.

5.	Data Query Language (DQL) commands:
o	SELECT: Retrieves data from one or more database tables.
o	JOIN: Combines rows from two or more tables based on a related column between them.

These are some of the common types of SQL commands. The specific commands available may vary depending on the SQL database management system (DBMS) you are using, as different DBMSs may have their own extensions and variations. 



# Defining SQL order of execution

## FROM Clause
SQL’s from clause selects and joins your tables and is the first executed part of a query. This means that in queries with joins, the join is the first thing to happen. 

It’s a good practice to limit or pre-aggregate tables before potentially large joins, which can otherwise be very memory intensive. Many modern SQL planners use logic and different types of joins to help optimize for different queries, which can be helpful but shouldn’t be relied on. 

In an instance like below, the SQL planner may know to pre-filter pings. That technically violates the correct SQL query order, but will return the correct result.

select
 count(*)
from
 pings
join
 signups
on
 pings.cookie = signups.cookie
where
 pings.url ilike '%/blog%'
However, if you are going to use columns in a way that prevents pre-filtering, the database will have to sort and join both full tables. For example, the following query requires a column from each table and will be forced into a join before any filtering takes place.

-- || is used for concatenation
select
 count(*)
from
 first_names
join last_names
 on first_names.id = last_names.id
where
 first_names.name || last_names.name ilike '%a%'
To speed up the query, you can pre-filter names with “a” in them:

with limited_first_names as (
 select
   *
 from
   first_names
 where
   name ilike '%a%'
)
, limited_last_names as (
  select
    *
  from
    last_names
  where
     name ilike '%a%'
)
select
 count(*)
from
 limited_first_names
join
 limited_last_names
on
 limited_last_names.id = limited_first_names.id
To learn more, you can also read about how we sped up our own queries by 50x using pre-aggregation.

 ## WHERE Clause
The where clause is used to limit the now-joined data by the values in your table’s columns. This can be used with any data type, including numbers, strings, or dates.

where nmbr > 5;
where strng = 'Skywalker';
where dte = '2017-01-01';
One frequent “gotcha” in SQL is trying to use a where statement to filter aggregations, which will violate SQL order of execution rules. This is because when the where statement is being evaluated, the “group by” statement has yet to be executed and aggregate values are unknown. Thus, the following query will fail:

select
 country
, sum(area)
from
 countries
where
 sum(area) > 1000
group by
 1
But it can be solved using the having clause, explained below.

## GROUP BY Clause

Group by collapses fields of the result set into their distinct values. This clause is used with aggregations such as sum() or count() to show one value per grouped field or combination of fields. 

When using group by: Group by X means put all those with the same value for X in the same row. Group by X, Y put all those with the same values for both X and Y in the same row.

The group by clause is worthy of its own post for many reasons, and you can find a lot more information about “group by” in other posts on our blog or in our whitepaper about SQL query order tips and a wide array of other tricks and best practices.

