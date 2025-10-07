# 16 Inserting Data

For 15 chapters, we've examined ways to read data using the `SELECT` statement. But all that data we read using the `SELECT` statement had to get into the tables somehow, so in this chapter, we'll learn how to insert data.

Throughout this book, we've seen that the syntax of SQL is often like the English language. When it comes to inserting rows of data, this similarity holds true because the new keyword we'll be using is `INSERT`. Let's look at some ways to use `INSERT` to add data to the tables in our database.

## 16.1 Inserting specific values

The first way to insert data is to use specific values. The main idea to keep in mind is that when we insert data, we're inserting a new row into an existing table.

Chapter 2 talked about rows, columns, and values. All tables in our relational database management system (RDBMS) have rows of data, and each row has a specific set of properties defined by the columns of the table. Each property in the columns is represented by some value of a particular data type, sometimes using NULL to represent the absence of a value for a particular column.

I hope that by now, the preceding paragraph makes total sense to you, especially given all the SQL queries you've written so far. If anything about it is unclear, I encourage you to review chapter 2 to solidify your understanding of rows, columns, and values. If everything makes sense, you can proceed to inserting some data.

### 16.1.1 Inserting a new row

As noted, we'll use `INSERT` as the keyword to insert data, and if we want to insert some specific values, we'll also use the keyword `VALUES`. For our first query, we'll insert a new row into the title table, but first, let's look at the data in that table.

In chapter 3, I warned you about using `SELECT` `*`, but because this table is small, you can use it to save yourself from typing all the column names in this ad hoc query. You'll use this logic a few times in this chapter to glance at the rows of data in tables (figure 16.1):

```sql
SELECT *
FROM title;
```

##### Figure 16.1 The results of all rows and columns in the title table

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH16_F01_Iannucci.png)

If we're going to insert a new row into the table shown in figure 16.1, we need to have values for all columns of data, and we need to use the correct data types. For readability, our query will specify the columns in the order in which they appear in the table: TitleID first, TitleName second, and so on.

Here's the query we'll use to insert a new row for the David Emptyfield title. We'll refer to this kind of query as an `INSERT` statement:

```sql
INSERT INTO title (
    TitleID,
    TitleName,
    Price,
    Advance,
    Royalty,
    PublicationDate
    )
VALUES (
    109,
    'David Emptyfield',
    9.95,
    0.00,
    10.00,
    '2022-01-16'
    );
```

In the preceding query, we're adding the next sequential TitleID (109), the TitleName, and the values for Price (9.95), Advance (0.00), Royalty (10.00), and PublicationDate (2022-01-16). If we execute this query, we should see a success message in the Output panel: 1 row(s) affected.

##### Try it now

Write and execute the preceding query. Typing all the column names of the title table may seem like a pain, but remember that you can use the Shift key to select all the column names in the Navigator panel and then drag them to your Query panel. If you don't recall how, see chapter 3 to refresh your memory.

Something else to notice about our query is that both the column names and the value names are surrounded by parentheses. When we use parentheses, we define the order of both the columns and values used by our `INSERT` statement.

Interestingly, we could just as easily have written the preceding query with a different column order. The only consideration would be that we'd need to rearrange the values in the same order in which the columns are specified in the `INSERT` statement.

Here's an example with the order of the columns reversed. If you executed the preceding `INSERT` statement, don't execute this:

```sql
INSERT INTO title (
    PublicationDate,
    Royalty,
    Advance,
    Price,
    TitleName,
    TitleID
    )
VALUES (
    '2022-01-16',
    10.00,
    0.00,
    9.95,
    'David Emptyfield',
    109
    );
```

Why would you change the column order for an `INSERT` statement? Well, normally, you wouldn't; that could confuse anyone who might read your SQL statement. If you find yourself inserting data into a table that has dozens or hundreds of columns, however, your SQL might be more readable if you organize column names alphabetically in your `INSERT` statement instead of in the order of the columns in the table. Apart from that example, you're probably better off ordering the column names in your `INSERT` statement in the same order as the columns of the table.

### 16.1.2 Inserting multiple new rows

Another interesting thing about enclosing values in parentheses is that the parentheses indicate all the values used for inserting a single row, which means we can also insert multiple rows if necessary. Just as we use a comma to indicate separate columns and values in our `INSERT` statement, we can use a comma to indicate separate sets of values to be inserted as rows. Here's an example of inserting two more rows into the title table using multiple values in the `VALUES` portion of our `INSERT` statement:

```sql
INSERT INTO title (
    TitleID,
    TitleName,
    Price,
    Advance,
    Royalty,
    PublicationDate
    )
VALUES (
    110,
    'Red Badge of Cursors',
    7.95,
    0.00,
    15.00,
    '2022-03-29'
    ),
    (
    111,
    'Of Mice and Metadata',
    8.95,
    0.00,
    12.00,
    '2022-05-17'
    );
```

This example notes only two additional rows, but there's no defined limit to how many rows you could insert into a table. Realistically, the only limitation is the amount of storage space in your database, which is difficult to determine with any kind of general SQL statement and thus is beyond the scope of this book. That said, unless you're inserting thousands or millions of rows, you probably don't have to worry about storage space.

##### Note

Speaking of general SQL statements, as you review SQL in another RDBMS, you may notice that someone else's SQL omits the word `INTO` from `INSERT` statements. That is, the SQL reads something like `INSERT [`table name`] VALUES (...)`. Omitting `INTO` is not optional in all RDBMSes, so I recommended that you get into the habit of including it in your code so you don't have to worry about syntax errors if you work with multiple RDBMSes.

So far, all the `INSERT` statements in this chapter involve inserting an entire row or rows of data. As you work with SQL, sometimes you'll need to insert values for every column but may be programmatically prevented from doing so. Let's look at how we can handle inserting a partial row.

### 16.1.3 Inserting a partial row

The term *partial row* may be confusing because I noted in chapter 2 that all rows in a given table must include values for all columns. This statement is still true, but two considerations allow us to insert a partial row of data.

The first consideration is when a table includes columns that allow null values. Let's look at the author table for an example (figure 16.2):

```sql
SELECT *
FROM author;
```

##### Figure 16.2 The results of all rows and columns in the author table

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH16_F02_Iannucci.png)

As we can see in figure 16.2, the MiddleName column allows null values because not all authors have a middle name. Although this nullable column presented challenges when we worked with functions and concatenation in chapter 15, it affords us the option to ignore it when inserting rows.

If we want to insert a row with a value of NULL for MiddleName, we can ignore the column in our `INSERT` statement. Doing this will result in a value of NULL by default. Here's an example:

```sql
INSERT INTO author (
    AuthorID,
    FirstName,
    LastName,
    PaymentMethod
    )
VALUES (
    12,
    'Whitney',
    'Miller',
    'Cash'
    );
```

We can execute this query without any error because the MiddleName column allows a value of NULL and enters it as a default for our new row. If we execute the preceding query and then select all the rows in the author table, we can verify this result (figure 16.3).

##### Figure 16.3 The results of all rows and columns in the author table, including the new row in which AuthorID is 12, which has a value of NULL for MiddleName

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH16_F03_Iannucci.png)

Not entering a value in this query may seem lazy, but you'll encounter tables in which you want to insert rows that have null values for certain columns. One common example is a table with columns for modification date and modification user to track when data was changed and who changed it. As we insert new data, we want those columns to be null to show that the data hasn't been modified since it was initially added to the table. Chapter 17 discusses modifying data in greater detail.

Getting back to entering partial rows, the second reason we may need to enter a partial row of data is that there are default constraints on one or more columns on a table. The architect of a table creates a *default constraint* to set a defined value for a column when data is inserted, which means that data for one or more columns is automatically populated instead of entered by our `INSERT` statement.

Let's look at the author table again. Notice that the first column is AuthorID and that it features an incremental sequence of numbers from 1 to 12. We need these numbers to be unique for each row because they represent the key values used to relate to data in other tables. (Chapter 8 talks about key values.)

Because these values need to be unique, database architects often put a default constraint on the first column. This constraint automatically populates that column with an identity value that uniquely identifies each row. This ensures that you and I don't accidentally enter the same identity value for different rows in the author table. If we did that, the relationships with data in any table using AuthorID would become ambiguous because they'd have relationships with one or more author rows.

Our author table currently has no default constraint for automatically populating the AuthorID column with a default value. We know this because we manually entered a value of 12 for the row we inserted. If a default constraint for an identity value were in place, we wouldn't be allowed to enter a value for AuthorID. In that case, which often occurs in database tables with ID values, we'd write our `INSERT` statement to omit the AuthorID column, like this:

```sql
INSERT INTO author (
    FirstName,
    MiddleName,
    LastName,
    PaymentMethod
    )
VALUES (
    'Whitney',
    NULL,
    'Miller',
    'Cash'
    );
```

Again, the database isn't set up with a default constraint on the author table, so don't execute this query. Just know that you'll need to account for these kinds of columns with partial inserts in many real-world databases.

### 16.1.4 A word of caution about omitting columns

You may see another kind of omission in the `INSERT` statement when you work with someone else's real-world SQL: the omission of the table columns in the `INSERT INTO` portion of the statement. You could write an `INSERT` statement like this one, for example, and execute it successfully:

```sql
INSERT INTO author
VALUES (
    12,
    'Whitney',
    NULL,
    'Miller',
    'Cash'
    );
```

This approach may be tempting, but if you consider what I've discussed in this chapter, you can already see why this option is dangerous:

* *This technique works only if you have values for all the columns in the table and have all the values in the correct order.* If you have one value too many or one value too few, the query will fail with an error. If you have the values in the wrong order, at best you'll have created a row with inaccurate data and at worst will encounter an error. (An execution error may be preferable to inaccurate data because you'd still have data integrity.)
* *This kind of query starts failing if you change the underlying table.* It's not uncommon to add columns to a table, and if you add a new column to the author table, this query will no longer execute successfully because you have more columns than values.
* *The table could have default constraints that prevent you from entering values for certain columns.* If the table had any default constraints, an `INSERT` statement without columns specified would fail to execute.

For all these reasons, you should always write `INSERT` statements that specify column names. If you see any SQL that doesn't, see whether you can rewrite it to include column names.

With that topic out of the way, let's move on to more options for inserting data. So far, we've used the `VALUES` keyword to insert rows of data, but we have other options. The good news is that you already know them.

## 16.2 Inserting a row with a query

When we use `VALUES` as we've used it in our `INSERT` statements thus far, we're telling the RDBMS, "Hey, RDBMS, I have these values, and I want to insert them into this table." In both SQL and English, our statement has two parts: the location of the insertion and the declaration of the values to be inserted.

To use a method instead of `VALUES` for our `INSERT` statement, we simply replace the latter part with a SQL query that has columns selected in an order and data type that match the columns of the table into which we're inserting, as defined in the first part of our query.

Because we've already inserted a new title (David Emptyfield) and a new author (Whitney Miller), we can relate this title to this author by using the titleauthor table. I haven't talked about this table yet, so let's look at it with an ad hoc `SELECT *` query (results shown in figure 16.4):

```sql
SELECT *
FROM titleauthor;
```

##### Figure 16.4 The results of all rows and columns in the titleauthor table

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH16_F04_Iannucci.png)

This table has three columns: TitleID, AuthorID, and AuthorOrder. As you may guess, TitleID relates to the TitleID column in the title table, and AuthorID relates to the AuthorID column in the author table. Notice that we don't necessarily have unique values for either column in this table. This table represents a many-to-many relationship because any title could have more than one author and any author could have contributed to more than one title.

The third column is AuthorOrder, which refers to the order of the authors as displayed on the cover of a title. The value in this column is always `1` for one author, which is what matters for the new row of data we're going to enter.

Let's start with a `SELECT` statement that allows us to query the required tables. Because this data isn't related yet, we can use subqueries as discussed in chapter 11:

```sql
SELECT
    (
    SELECT TitleID
    FROM title
    WHERE TitleName = 'David Emptyfield'
    ) AS TitleID,
    (
    SELECT AuthorID
    FROM author
    WHERE FirstName = 'Whitney'
        AND LastName = 'Miller'
    ) AS AuthorID;
```

##### Note

In chapter 11, I noted that using subqueries in `SELECT` statements is rarely the best idea, but in this `INSERT` statement, using a subquery makes sense because it's an easy way to add these values to the titleauthor table.

This query serves as the starting point for our insert. Note that although column aliases are not required, they've been added for readability and to help us match the resulting values to the columns in the titleauthor table.

Let's add the necessary parts to make this query an `INSERT` statement, including adding a value of `1` for the AuthorOrder column. Here's what our final query will look like:

```sql
INSERT INTO titleauthor (
    TitleID,
    AuthorID,
    AuthorOrder
    )
SELECT
    (
    SELECT TitleID
    FROM title
    WHERE TitleName = 'David Emptyfield'
    ) AS TitleID,
    (
    SELECT AuthorID
    FROM author
    WHERE FirstName = 'Whitney'
        AND LastName = 'Miller'
    ) AS AuthorID,
    1 AS AuthorOrder;
```

Again, the columns in the `SELECT` clause are aliased for readability. But the important part is that we have the columns in both the `INSERT` and `SELECT` parts of the query in matching order.

##### Try it now

If you haven't already done so, execute the SQL that inserts David Emptyfield into the title table (section 16.1.1) and Whitney Miller into the author table (section 16.1.3), followed by the `INSERT` statement that relates them in the titleauthor table. You'll use this data in chapter 17.

## 16.3 Inserting a row with variables

Now let's look at another way to insert data: using variables. In chapter 13, we worked with many `SELECT` queries that used variables, which are highly desirable for any kind of repeatable SQL we need to use. We can declare and use variables in an `INSERT` statement that uses `SELECT` to make easily repeatable SQL that needs only the variables changed. Here's an example that we can use to add A Table of Two Cities to the title table:

```sql
SET
    @TitleID = 112,
    @TitleName = 'A Table of Two Cities',
    @Price = 9.95,
    @Advance = 0.00,
    @Royalty = 15.00,
    @PublicationDate = '2022-08-07';

INSERT INTO title (
    TitleID,
    TitleName,
    Price,
    Advance,
    Royalty,
    PublicationDate
    )
VALUES (
    @TitleID,
    @TitleName,
    @Price,
    @Advance,
    @Royalty,
    @PublicationDate
    );
```

This example is only one way to use variables to insert data. If you've read chapter 13, I'm sure that you're already thinking of other ways to use variables to add data to tables in a database.

By learning how to add data with `INSERT` statements, you've started executing SQL statements that are categorized as data manipulation. *Data manipulation* refers to the process of changing data, and it applies not only to adding data but also to modifying or even removing data. In chapter 17, you'll expand your data-manipulation skills by learning how to modify and remove data.

## 16.4 Lab

1.  Will this SQL, which has different alias column names from the table column names, execute successfully?

```sql
INSERT INTO titleauthor (
    TitleID,
    AuthorID,
    AuthorOrder
    )
SELECT
    (
    SELECT TitleID
    FROM title
    WHERE TitleName = 'David Emptyfield'
    ) AS TID,
    (
    SELECT AuthorID
    FROM author
    WHERE FirstName = 'Whitney'
        AND LastName = 'Miller'
    ) AS AID,
    1 AS AO;
```

2.  Will this SQL, with no table column names specified, execute successfully?

```sql
INSERT INTO author
VALUES (
    13,
    'Jeff',
    'Iannucci'
    );
```

3.  Insert a new row into the promotions table, using the following values:

•   PromotionID—13

•   PromotionCode—2OFF2022

•   PromotionStartDate—May 1, 2022

•   PromotionEndDate—May 15, 2022

4.  Insert a new row into the customer table for Gianluca Rossi. The CustomerID should be 21, but all other information will be the same as that of Mia Rossi, whose CustomerID is 20. For this exercise, you should use `SELECT` instead of `VALUES` for the insert.

## 16.5 Lab answers

1.  Yes. The column names, whether or not they're noted in the aliases in the `SELECT` clause, don't need to match the column names in the table where the data is being inserted. What matters is that you have the same number of columns with matching data types.

2.  No. Execution of this query will result in an error message indicating that the column count doesn't match because the author table has four columns, and this query is attempting to insert values for only three values. This result is one reason why you should always specify the columns in the table into which the data is being inserted.

3.  Your code should look something like this:

```sql
INSERT INTO promotion (
    PromotionID,
    PromotionCode,
    PromotionStartDate,
    PromotionEndDate
    )
VALUES (
    13,
    '2OFF2022',
    '2022-05-01 00:00:00',
    '2022-05-15 00:00:00'
    );
```

4.  Although you could specify all the values being inserted for the new row with literal values, you could also copy the existing values where CustomerID=20 except CustomerID and FirstName, like this:

```sql
INSERT INTO customer (
    CustomerID,
    FirstName,
    LastName,
    Address,
    City,
    State,
    Zip,
    Country
    )
SELECT
    21,
    'Gianluca',
    LastName,
    Address,
    City,
    State,
    Zip,
    Country
FROM customer
WHERE CustomerID = 20;
```

