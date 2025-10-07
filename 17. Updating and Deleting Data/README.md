# 17 Updating and Deleting Data

Chapter 16 discussed inserting new rows of data into tables and contained our first exercises for doing something with SQL other than reading data. The chapter also briefly mentioned that the `INSERT` keyword is one of several used for data manipulation.

This chapter examines two other ways of manipulating data: updating and deleting. Because SQL is designed to be intuitive for English speakers, we'll work with the keywords `UPDATE` and `DELETE`, respectively.

## 17.1 Updating values

Updating data is a bit different from inserting data, in that we're manipulating data at the column level instead of the row level. Recall that tables have rows, and rows have properties represented by columns. When we update data in SQL, we're updating the values of those properties, so we're making changes at the column level.

These changes may or may not include all columns in a given row, and they may or may not involve updating the values of one or more columns of every row in the table. The point is that we have many options for updating data in SQL. If this discussion is confusing, some examples may help you understand the options.

### 17.1.1 Working with data manipulation in real time

Before we begin, I want to draw your attention to a feature of SQL and relational databases that often catches people off guard. You may be accustomed to working with a word processor, a spreadsheet, or another application that contains some kind of data and saves your changes only when you click a particular button or press a certain series of keys. In these applications, if you make a mistake, you can always go back and undo the changes before you save them.

Unlike in those applications, any data manipulation you make in a relational database happens in real time and is permanent. If you have an "oops" moment, you have no way to undo the changes; they are committed instantly. This fact may seem alarming, but it's necessary to enable a relational database management system (RDBMS) to provide up-to-date data quickly and accurately to hundreds or thousands of users who are connected to and continually querying a database.

##### Warning

It's important to understand that every data change we'll make in this chapter will happen in real time. The RDBMS has no Save button or keystroke.

Because these changes happen instantly and humans have a knack for making mistakes, the MySQL Workbench has a built-in safety feature called Safe Updates. Safe Updates is enabled by default, and it reduces the chance of an accidental data change that can affect every row in a table.

With Safe Updates enabled, for example, we can't make any updates that don't specify the key value of a table. Because our tables currently don't have key values, we need to disable this feature to make updates. If we don't, our SQL statements in this chapter will result in errors.

You have a few ways to disable this feature, but the most complete way is to choose Edit > Preferences in the top-left corner of the MySQL Workbench. This command opens the Workbench Preferences dialog box, shown in figure 17.1. Select SQL Editor in the pane on the left side; then clear the Safe Updates check box in the main pane. After clearing the check box, click OK to close the window, and then close and restart Workbench. Now that Safe Updates is disabled, we can begin (carefully) to update our data.

##### Figure 17.1 Safe Updates disabled in the Workbench Preferences dialog box

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH17_F01_Iannucci.png)

### 17.1.2 Requirements for updates

For our first example, we want to update the price of a single title, Pride and Predicates, from $9.95 to $8.95. As in so many statements in SQL, let's start with an English verbal declaration: "I would like to update the price of Pride and Predicates to $8.95."

This statement is a good starting point, and it's close to what the SQL will be, but we have to make a few changes to account for table and column names: "I would like to update the price to $8.95 where the title name is Pride and Predicates."

Our verbal statement is closer to the SQL. We have our filter ("where the title name is Pride and Predicates"), but we need to mention the table that contains the data. We do this by saying we want to update the table and then setting one or more column values to some other value: "I would like to update the title table. I would like to set the price to $8.95 where the title name is Pride and Predicates."

This verbal statement is perfect. Let's convert it to a SQL statement:

```sql
UPDATE title
SET Price = 8.95
WHERE TitleName = 'Pride and Predicates';
```

We have one new keyword, `UPDATE`, and one we've used with variables, `SET`. We use `UPDATE` to indicate the table in which data is to be updated, and we use `SET` to assign new values much the same way that we did when assigning values to variables. This `UPDATE` statement highlights the three requirements for any update and the order they must be in to create a valid SQL statement:

1. The table to be updated, using `UPDATE`
2. The column name(s) and new values to be assigned, using `SET`
3. The filter condition to determine which rows are to be updated, using `WHERE` in this case

##### Warning

Although it's technically not required, the `WHERE` clause is perhaps the most critical. If you don't write *and execute* the filtering condition, you'll update *every* row in the table. Because these changes are made in real time, always take the utmost care to write and execute the filter to change only the values in the intended rows.

### 17.1.3 Updating values in one or more columns

As you may have noticed in the second requirement for update statements, you can update values in one or more columns. Much as you use commas to indicate multiple columns in a `SELECT` statement, you can use commas to indicate two or more desired updates in an `UPDATE` statement.

##### NOTE

All values to be changed in an `UPDATE` statement must be intended for the same table and meet the requirements of the filtering condition.

This example updates the Advance and Royalty values to 0 and 10, respectively, for the title Pride and Predicates:

```sql
UPDATE title
SET Advance = 0.00,
    Royalty = 10.00
WHERE TitleName = 'Pride and Predicates';
```

Although we're using the TitleName in our filtering condition, it's more common to use key values or some other unique identifier for `UPDATE` statements to ensure that we're updating the precise row or rows intended. Let's change our `UPDATE` statement to use the identifier TitleID, which for Pride and Predicates is 101:

```sql
UPDATE title
SET Advance = 0.00,
    Royalty = 10.00
WHERE TitleID = 101;
```

##### Note

If you executed the preceding two statements, you noticed that both of them succeeded but displayed different messages in the Output panel. Whereas the former statement displays "1 row(s) affected" and "Changed: 1," the latter statement displays "0 row(s) affected" and "Changed: 0." There was nothing to change with the second `UPDATE` statement because the Advance and Royalty values were already set to the intended values with the execution of the first `UPDATE` statement.

We can also use variables to create repeatable code as we did in chapter 16 with `INSERT` statements. In this example, we set all the values back to what they originally were for Pride and Predicates, using TitleID as the filter condition:

```sql
SET
    @TitleID = 101,
    @Price = 9.95,
    @Advance = 5000.00,
    @Royalty = 15.00;

UPDATE title
SET Price = @Price,
    Advance = @Advance,
    Royalty = @Royalty
WHERE TitleID = @TitleID;
```

##### Try it now

If you haven't already done so, execute the `UPDATE` statements you've seen so far in this chapter. Use a query such as `SELECT` `*` `from` `title` `WHERE` `TitleID` `=` `101;` between the statements to verify that the changes occurred.

We can make one other change with an update: remove a value. Recall that every column for every row of a table in an RDBMS must have some representation of a value, so the only option we have to remove a value is to set it to NULL, which indicates the absence of a value.

Not every column allows null values, but we know at least one column in our database that does: the MiddleName column in the author table. If we want to set the MiddleName for the first author to NULL, we'd write an `UPDATE` statement like this:

```sql
UPDATE author
SET MiddleName = NULL
WHERE AuthorID = 1;
```

These are our options for updating values with a basic `UPDATE` statement. Next, let's look at how to use an `UPDATE` statement to query multiple tables as part of our filtering condition.

### 17.1.4 Updating values with a multitable query

In chapter 8, I noted that a *predicate* is any part of a SQL statement that evaluates whether something is true, false, or unknown. Since that chapter, we've used predicates for filtering conditions, which included conditions in the `FROM`, `WHERE`, and `HAVING` clauses.

As you work with SQL, you'll likely need to write an `UPDATE` statement that requires filtering with a predicate that uses more than one table. This kind of SQL statement can be tricky because we can update values in only one table in an `UPDATE` statement, and we want to make sure that we do it correctly. We need a `FROM` clause to identify the filtering conditions, and we need to put it in the correct place in our SQL statement.

Unfortunately, we can put this predicate in multiple places in an `UPDATE` statement, depending on which RDBMS we're using. Although many uses of keywords are standardized for all RDBMSes, no standard exists for updating values in a query with multiple tables.

##### WARNING

Although this section contains examples of updating values in some popular RDBMSes other than MySQL, these examples are not comprehensive. Be sure to check the documentation of the RDBMS you use to confirm the correct syntax for these kinds of `UPDATE` statements.

Suppose that we want to update the price of any titles by a particular author in our sqlnovel database in MySQL. We know the AuthorID, which is 12, but we don't have the TitleName or TitleID of any particular title. We also want to update the price to $8.95. To complete this update, we need to join at least two tables.

As a refresher, the relationship between the title and author tables is established through the titleauthor table. Rows are uniquely identified in the title table by TitleID and in the author table by AuthorID. The titleauthor table contains both TitleID and AuthorID columns in a many-to-many relationship because any author could write more than one title and any title could have more than one author.

Before we dive into the `UPDATE` statement, let's start with a `SELECT` query to see the value we intend to update. If we want to write a query that shows the Price for any titles by the author with the AuthorID of 12, we might write a query using joins in the `FROM` clause like this:

```sql
SELECT title.Price
FROM title
INNER JOIN titleauthor
    ON title.TitleID = titleauthor.TitleID
INNER JOIN author
    ON titleauthor.AuthorID = author.AuthorID
WHERE author.AuthorID = 12;
```

This query returns the value we intend to change, but we can make it a bit more readable by using table aliases, as discussed in chapter 8. Let's use some table aliases to reduce the wordiness of our query:

```sql
SELECT t.Price
FROM title t
INNER JOIN titleauthor ta
    ON t.TitleID = ta.TitleID
INNER JOIN author a
    ON ta.AuthorID = a.AuthorID
WHERE a.AuthorID = 12;
```

There—that's a bit easier to read. Savvy readers may also note that we don't need the author table in this query because the AuthorID value is also included in the titleauthor table. We can make our query simpler by removing any reference to the author table and filtering on AuthorID in the titleauthor table:

```sql
SELECT t.Price
FROM title t
INNER JOIN titleauthor ta
    ON t.TitleID = ta.TitleID
WHERE ta.AuthorID = 12;
```

Now that we have the foundation of a `SELECT` statement, we can easily change it to our desired `UPDATE` statement by removing the `SELECT` clause and replacing the `FROM` keyword with `UPDATE`. Then, after the `UPDATE`, we can use `SET` to set the values before the `WHERE` clause. Because we established table aliases in the preceding `FROM` clause, we can use the same table aliases in our query to indicate which table in our query is having data updated. Here's what our `UPDATE` statement will look like:

```sql
UPDATE title t
INNER JOIN titleauthor ta
    ON t.TitleID = ta.TitleID
SET t.Price = 8.95
WHERE ta.AuthorID = 12;
```

This example may be a bit confusing because we split the predicate into two separate parts of our query, with the joining of tables in the update before the `SET` and the rest of the filtering after the `SET` in the `WHERE` clause. Again, this method of updating is specific to MySQL.

##### Note

The following examples are for common RDBMSes other than MySQL. These examples are provided to show differences and are not intended to be executed in MySQL. If you attempt to execute them, they'll result in failed queries and error messages.

If our database used PostgreSQL, we'd still indicate the table to be updated and any alias in the `UPDATE` portion of our query, but we'd use the `FROM` clause to join any additional tables. The following query works in PostgreSQL but not MySQL:

```sql
UPDATE title t
SET t.Price = 8.95
FROM titleauthor ta
WHERE t.TitleID = ta.TitleID
    AND ta.AuthorID = 12;
```

If our database was in a SQL Server instance, the logic might make a bit more sense compared with the `SELECT` query. In SQL Server, we'd simply replace the `SELECT` with the `UPDATE` and `SET` parts and leave the `FROM` and `WHERE` clauses unchanged. The following query works in SQL Server but not in MySQL:

```sql
UPDATE title t
SET t.Price = 8.95
FROM title t
INNER JOIN titleauthor ta
    ON t.TitleID = ta.TitleID
WHERE ta.AuthorID = 12;
```

The point is that although you can use joins for filtering in an `UPDATE` statement, each RDBMS has its own syntax for these kinds of queries. Unfortunately, this example is one of those rare parts of learning SQL that require you to venture beyond this book to learn the syntax of the RDBMS you're using.

## 17.2 Deleting rows

When we delete data using SQL, we're effectively doing the opposite of what an `INSERT` statement does. Whereas `INSERT` adds one or more rows to a table, `DELETE` removes one or more rows. Also, as with `INSERT` and `UPDATE` statements, we can change data in only one table per `DELETE` statement.

### 17.2.1 Deleting one or more rows

As we've done so many times, let's start with a verbal declaration. Suppose that we want to delete the row from the title table in which TitleID = 110. We might start with a verbal declaration like this: "I would like to delete any rows from the title table where the TitleID is 110." Our SQL statement is very close to this statement, using a new keyword, `DELETE:`

```sql
DELETE
FROM title
WHERE TitleID = 110;
```

As with `INSERT` and `UPDATE` statements, we're starting our statement with a data-manipulation keyword, which in this case is `DELETE`. Next, we identify the table from which we intend to delete data. Finally, we indicate the filtering to be used so that we delete only the intended rows.

##### Warning

As with `UPDATE` statements, the `WHERE` clause isn't required but is perhaps the most critical part of our query. If you don't write *and execute* this filtering condition, you'll delete every row in the table. As you can imagine, this result can be catastrophic. Because these changes occur in real time, always take the utmost care to write and execute the filter to delete only the intended rows.

We can also use variables, of course, to make repeatable SQL statements for a query like this example. Here's how we'd do this for the preceding query:

```sql
SET @TitleID = 110;

DELETE
FROM title
WHERE TitleID = @TitleID;
```

##### Tip

Some RDBMSes allow the omission of the `FROM` keyword in `DELETE` statements like these, but you should use it anyway. Although it's best to reduce the wordiness of queries whenever possible, omitting one word doesn't reduce wordiness significantly. Moreover, if you migrate your query from one RDBMS to another later, omitting the word `FROM` could result in a query that doesn't work.

### 17.2.2 Deleting a row with a multitable query

As discussed in section 17.1.4, the syntax for an `UPDATE` statement that joins multiple tables in the predicate varies depending on the RDBMS you're using. Unfortunately, the syntax for a `DELETE` statement can also vary from one RDBMS to another. The good news is that the syntax doesn't vary quite as much because MySQL, SQL Server, and MariaDB have the same syntax.

Even better news: this syntax is remarkably similar to a `SELECT` statement. In section 17.1.4, we created this `SELECT` statement before writing SQL for an `UPDATE` statement for a title that related to AuthorID 12:

```sql
SELECT t.Price
FROM title t
INNER JOIN titleauthor ta
    ON t.TitleID = ta.TitleID
WHERE ta.AuthorID = 12;
```

If we want to delete the row in the title table instead of updating the value for Price, we'd simply replace the `SELECT` clause with a `DELETE` that noted the table with rows to be deleted, like this:

```sql
DELETE t
FROM title t
INNER JOIN titleauthor ta
    ON t.TitleID = ta.TitleID
WHERE ta.AuthorID = 12;
```

Again, this syntax works in MySQL and a few other RDBMSes, but not all. Please refer to the documentation of the particular RDBMS you're using for the correct syntax to delete rows using a predicate that joins multiple tables.

### 17.2.3 Deleting all rows in a table

As noted in the warning in section 17.2.1, if a `DELETE` statement is executed without a `WHERE` clause, it can remove all the rows from a table. I say "can" because as mentioned in chapter 16, databases are often designed with certain constraints, including those that allow or prevent the insertion of certain values into a table. Those constraints can allow or prevent the deletion of rows as well.

If we don't have these constraints, and the tables in our database currently don't, we can remove all the rows in a table by executing a `DELETE` statement that has no filtering condition. We don't want to delete all the rows from the tables in our database, but for the sake of practice, we can execute this query on the myfirstquery table we used in chapter 2 with minimal effect on any future exercises. Here's what that query would look like:

```sql
DELETE
FROM myfirstquery;
```

As expected, executing this query results in the deletion of all rows in the myfirstquery table. We also have a more efficient way to remove all rows from a table, a way that's available in nearly every RDBMS: `TRUNCATE` `TABLE`.

Unlike removing all rows with a `DELETE` statement, which scans every row in a table and deletes them one at a time, `TRUNCATE` `TABLE` deletes rows without scanning them individually. This statement is designed for speed.

The syntax for `TRUNCATE` `TABLE` is simple. If we want to truncate the myfirstquery table, our SQL statement would look like this:

```sql
TRUNCATE TABLE myfirstquery;
```

The `TRUNCATE` `TABLE` statement has a lot less flexibility than a `DELETE` statement because it's designed for the single purpose of removing all rows from a table quickly. It can't be used to remove one row or some rows. Also, the same problems with constraints that can prevent a `DELETE` statement from deleting rows can prevent `TRUNCATE` `TABLE` from executing successfully.

##### Try it now

Use the preceding `DELETE` and `TRUNCATE TABLE` statements to remove all the rows from the myfirstquery table. If you want to test deleting multiple rows, try executing one or more `INSERT` statements to add rows to the myfirstquery table.

## 17.3 One big tip for data manipulation

In section 17.1.4, we wrote a `SELECT` statement to find the data to update. Although we did this to compare the basic structure of the `SELECT` statement with our `UPDATE` statement, we could just as easily have used this `SELECT` statement to validate which rows were going to be updated.

Two warnings in this chapter caution against accidentally updating or deleting all data in a table. I could have included even more warnings about updating the wrong data through incorrect logic for filtering contained in the predicates of `UPDATE` and `DELETE` statements. Remember that all changes occur in real time, so use caution when updating and deleting data.

One significant and common way to exercise caution is to write and execute a `SELECT` statement that uses the same logic contained in an `UPDATE` or `DELETE` statement. Doing this allows you to see what data will be affected before you execute your data-manipulation statement. I can tell you from experience that this approach—selecting data through a query first—has saved me and countless others from catastrophic results.

Although there's no way to undo an executed query, you can always proactively review the affected rows using a `SELECT`. I highly recommend selecting data with your filtering conditions, no matter how experienced you become in writing SQL.

## 17.4 Lab

1.  Using what you learned in chapter 16, write and execute a query to add your name and address to the customer table. Use the value 22 for CustomerID. (Hint: You can drag and drop column names from the Navigator panel in MySQL Workbench.)

2.  Write and execute a query to update the address of the row you just added to an address at which you previously resided (or any other address).

3.  Write and execute a query to delete the row you inserted and updated in the customer table, but use a `SELECT` first to confirm that your SQL statement will produce the correct results.

## 17.5 Lab answers

1.  Your query to insert a row into the customer table might look something like this:

```sql
INSERT customer (
    CustomerID,
    FirstName,
    LastName,
    Address,
    City,
    State,
    Zip,
    Country
)
VALUES (
    22,
    'Jeff',
    'Iannucci',
    '1600 Pennsylvania Ave NW',
    'Washington',
    'DC',
    '20500',
    'USA'
    );
```

2.  Your query to update your address in the customer table might look something like this:

```sql
UPDATE customer
SET Address = '1700 W Washington St',
    City = 'Phoenix',
    State = 'AZ',
    Zip = '85007'
WHERE CustomerID = 22;
```

3.  Your query to delete your row from the customer table might look something like this:

```sql
DELETE
FROM customer
WHERE CustomerID = 22;
```

