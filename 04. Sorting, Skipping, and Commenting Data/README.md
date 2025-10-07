# 4 Sorting, Skipping, and Commenting Data

In chapter 3, I noted that your relational database management system (RDBMS) won't return results in a predictable order. This is by design, as any given request may or may not need the data to be ordered, and the RDBMS is simply taking the most efficient way of returning the data requested. The results may appear in some sort of order, such as the order in which the rows were added to a table, but there are no implicit guarantees about how the data in the results of a query will be ordered. If we want to be certain of the order of results, we need to say so explicitly in our SQL query.

Some fun features associated with ordering data allow us to manipulate the number of rows returned in our result set. These features can be useful if we have millions or billions of rows and need to see only the most recent entries or the entries with the smallest or largest values.

Because you're just starting to write SQL, this chapter also discusses how to use comments in SQL. Comments are indispensable tools for you and anyone else who may read your SQL, and if you're going to write queries properly, you should start building the habit of using comments today.

## 4.1 Sorting data

Everywhere we go, we find things sorted in some order. Books in a library are sorted by author name, floors in a building are sorted by number, and events in a daily planner are sorted by date and time. All kinds of things in our lives are organized for ease of use and readability, and the data we use should be no different.

### 4.1.1 Sorting by one column

To see how we do this in SQL, let's go back to a declarative sentence from chapter 3: "I would like all the TitleNames and Prices of the titles." The SQL for this request is as follows:

```sql
SELECT
    TitleName,
    Price
FROM title;
```

As we can see in figure 4.1, this query won't return the data in a particular way.

##### Figure 4.1 The results of the query of TitleName and Price from the title table. The results are not ordered by either TitleName or Price.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH04_F01_Iannucci.png)

Let's declare that we want the same results, this time ordered by the title name: "I would like all the TitleNames and Prices of the titles, and I would like the results ordered by TitleName." As before, the SQL we will use is very similar to what we just said. To fulfill the new request, we will add a new clause using the `ORDER` `BY` keyword:

```sql
SELECT
    TitleName,
    Price
FROM title
ORDER BY
    TitleName;
```

When we execute this command, we see that the rows returned are the same as before, but now they are ordered as requested, as shown in figure 4.2.

##### Figure 4.2 The results of the preceding query, with the results now sorted alphabetically by TitleName

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH04_F02_Iannucci.png)

When it's used, the `ORDER` `BY` clause should almost always be the last clause in a SQL statement. With only one exception (which we'll get to later in this chapter), specifying any other clause after the `ORDER` `BY` clause in a query will result in a syntax error.

This should be easy to remember if you consider that data ordering will be the last operation the RDBMS will perform on your data. This point is often misunderstood; too many people use SQL thinking that `ORDER` `BY` indicates how the data will be read. In reality, the RDBMS has to complete the operations for the rest of your SQL statement first; then, after getting your result set, it organizes the data the way your query requests in the `ORDER BY` clause.

##### Warning

The additional work that `ORDER` `BY` requires of the RDBMS is minimal for the queries in this book because the result sets are only a handful of rows. When you start querying data sets with millions or even billions of rows, however, adding an `ORDER` `BY` can be catastrophic for query performance. Even more problematic is the fact that ordering data for very large data sets can require large amounts of resources from the computer's processor and memory, resulting in degraded performance for queries that other users are running. Always be careful with your use of `ORDER` `BY`.

When you consider that your RDBMS has to gather the entire result set before sorting the results, it should make sense that you can even use a column alias in the `ORDER BY` clause. The column aliases are applied when your result set has been created and before it has been ordered, so SQL logic like the following also works for sorting data, as shown in figure 4.3:

```sql
SELECT
    TitleName AS NameOfTheBook,
    Price
FROM title
ORDER BY
    NameOfTheBook;
```

##### Figure 4.3 After you use an ORDER BY, the results of the preceding query are ordered by TitleName, which has been aliased as NameOfTheBook.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH04_F03_Iannucci.png)

### 4.1.2 Sorting by multiple columns

`ORDER BY` isn't limited to a single column, of course. We can use commas in the `ORDER BY` clause similarly to the way we use them in the `SELECT` clause to specify ordering by multiple columns. Consider the following query:

```sql
SELECT
    TitleName,
    Advance,
    Royalty
FROM title
ORDER BY
    Advance,
    Royalty;
```

As figure 4.4 shows, the results of our query are ordered primarily by the Advance column, from lowest to highest. But when the values in Advance are the same—as they are for the third, fourth, and fifth rows, all with a value of 5000.00—the Royalty column is used to order those three rows.

##### Figure 4.4 The results of a query ordered by the Advance column and then by the Royalty column, both from highest value to lowest

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH04_F04_Iannucci.png)

##### Try it now

Now that you've seen a few ways to sort data in the title table, try using `ORDER` `BY` as shown in the examples, or try sorting on other columns, such as Price or PublicationDate.

### 4.1.3 Specifying sort direction

When we sort data with `ORDER` `BY`, there is an implicit direction of sorting, either alphabetically from A to Z or numerically from lowest to highest value. This direction is known as *ascending* data order. We can also sort results in the other direction, from Z to A or from highest to lowest values. This direction is known as *descending* order, and we must state it explicitly by adding `DESC` after the column name in the `ORDER BY` clause. Here's an example of the previous query data sorted by Advance values in descending order:

```sql
SELECT
    TitleName,
    Advance,
    Royalty
FROM title
ORDER BY
    Advance DESC,
Royalty;
```

The results in figure 4.5 show that the results are now ordered by Advance values from largest to smallest. But take a closer look at the fourth, fifth, and sixth rows: they are still sorted in value from smallest to largest Royalty. Royalty is still sorted in ascending order because no order was specified for this column.

##### Figure 4.5 The results are now sorted by Advance descending and Royalty ascending.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH04_F05_Iannucci.png)

For clarity, we can explicitly state the sort order for the Royalty column as well by adding `ASC` for ascending sort order to the `ORDER` `BY` clause, like this:

```sql
SELECT
    TitleName,
    Advance,
Royalty
FROM title
ORDER BY
    Advance DESC,
    Royalty ASC;
```

##### TIP

The implicit nature of ascending order in an `ORDER` `BY` column can be confusing, so when you're writing SQL that other people will read, get into the habit of explicitly stating the direction of ordering, even though you don't need to for ascending order. As noted earlier, you should always try to make your SQL as clear and easy to understand as possible.

### 4.1.4 Sorting by hidden columns

You may encounter a certain scenario in which you want to order the results by a column that you don't want returned in the result set. This scenario is possible because you can order your results by one or more columns that aren't seen. Suppose that you revise the preceding query to return only the TitleName but still order the results by the Advance (descending) and Royalty (ascending) columns:

```sql
SELECT
    TitleName
FROM title
ORDER BY
    Advance DESC,
    Royalty ASC;
```

The results in figure 4.6 are the same as for the preceding query except that the Advance and Royalty columns aren't returned.

##### Figure 4.6 Only the TitleName values are in the results, but the rows are still sorted by Advance descending and Royalty ascending.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH04_F06_Iannucci.png)

How can we sort by data that isn't included in our `SELECT`? The RDBMS accomplishes this little bit of magic by adding Advance and Royalty to the result set before it's returned to you, then organizing the data as requested, and finally returning only the columns requested in the `SELECT` clause. As you can imagine, this process is an extra bit of work, so be careful when you use this technique with very large data sets.

### 4.1.5 Sorting by position

If column names seem too long to type, you have a quicker way to specify sort order: list the *numerical column position* in the `SELECT` clause instead of the column name. Think of the numerical order of the columns in the `SELECT` clause, TitleName (1), Advance (2), and Royalty (3):

```sql
SELECT
    TitleName,
    Advance,
    Royalty
FROM title
ORDER BY
    2 DESC,
    3 ASC;
```

Now the sort order is listed as Advance descending and Royalty ascending. We know this because we can determine that Advance is the second column, represented by the `2` value in the `ORDER` `BY`, and Royalty is the third column, represented by the `3` value.

##### WARNING

This kind of shorthand notation in the `ORDER` `BY` may be useful when you're writing SQL quickly for ad hoc queries, but because the readability is inferior to explicitly naming columns to be sorted, you should avoid this technique in any reusable SQL you write. As you can imagine, if the columns in the `SELECT` change, ordering by position would use different columns.

## 4.2 Skipping data

The result set of every query we've run includes all the data in the table. What if you don't want all the data returned? On some occasions, you want only a handful of rows to survey or maybe just one, skipping most of the result set. You may need to look at a table of data you're unfamiliar with to see how the data is formatted, for example. You can certainly do this in SQL.

### 4.2.1 Using LIMIT to reduce results

Let's use our declarative English language first to state our intentions of finding just three published books: "I would like all the TitleNames and PublicationDates, but limit the results to the first three rows." The new keyword here is `LIMIT`, which will be used to reduce the result set to a specified number of rows. We can accomplish this by using `LIMIT` like this:

```sql
SELECT
    TitleName,
    PublicationDate
FROM title
LIMIT 3;
```

The RDBMS grabs the first three rows it can find, so your results should look something like figure 4.7.

##### Figure 4.7 The results of using LIMIT to return only three rows

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH04_F07_Iannucci.png)

Using `LIMIT` with a result set returns only three rows instead of all eight in the table. Although this command may not seem useful, it can be incredibly helpful if you want to quickly sample the column names and types of values they contain.

##### Try it now

Using `SELECT *` and `LIMIT`, write a query that allows you to quickly sample some rows in the title table. You may or may not get rows with the three titles shown in figure 4.7, but that's because you didn't specify any sort order.

Because the preceding query would be used to sample data, the results can be imprecise. Let's use our declarative English language first to state our intentions of finding something more precise, namely the three most recently published books: "I would like all the TitleNames and PublicationDates, but limit the results to the three most recent PublicationDates."

To execute this query, we'll bring back `ORDER BY`, sorting by PublicationDate descending to give us rows with the most recent (latest) date values:

```sql
SELECT
    TitleName,
    PublicationDate
FROM title
ORDER BY PublicationDate DESC
LIMIT 3;
```

Note the order of the clauses here: the `LIMIT` clause is after the `ORDER` `BY`. The `LIMIT` clause is the only clause that should ever follow the `ORDER BY` clause, and if it's included, it's always the last clause in a SQL query. Placing the `LIMIT` clause anywhere else will result in a syntax error. Figure 4.8 shows the results of this query, which returns the three most recent TitleName values and their PublicationDate values.

##### Figure 4.8 The three most recently published TitleNames and their PublicationDates when you use ORDER BY on PublicationDate and LIMIT the results to three rows

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH04_F08_Iannucci.png)

##### TIP

Although you're not required to do so, you'll almost always want to use `ORDER BY` whenever you use the `LIMIT` clause. Why? You'll likely intend to read a limited sample of rows based on their being the oldest, newest, largest, smallest, or some other order. As noted, using `LIMIT` without specifying the order can return random and unpredictable results.

### 4.2.2 Using OFFSET to select a different limited set

The scenario of writing a SQL statement to find the most recent data is not uncommon, but at times, you may want to skip certain rows other than the most or least. In those cases, you can use another feature of the `LIMIT` clause. The way to do this is to use an additional option in the clause: `OFFSET`. `OFFSET` can't be executed without `LIMIT`, but it can direct the RDBMS to ignore a specified number of rows before it starts returning the rows indicated by the `LIMIT` clause. Let's rerun the preceding query but use `OFFSET` to skip the first row that would be returned:

```sql
SELECT
    TitleName,
    PublicationDate
FROM title
ORDER BY PublicationDate DESC
LIMIT 3 OFFSET 1;
```

Figure 4.9 shows that the row with TitleName "The Sum Also Rises" has been skipped, and the results include a different row with TitleName "The DateTime Machine," which has an older PublicationDate.

##### Figure 4.9 The most recent three TitleName and PublicationDate values after you use OFFSET to skip the first row

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH04_F09_Iannucci.png)

### 4.2.3 Limiting data in another RDBMS

Chapter 1 discussed the fact that each RDBMS has its own variation for certain commands. Unfortunately, the `LIMIT` clause is one of those commands.

##### Warning

The `LIMIT` clause works with many popular RDBMSs, including MySQL, MariaDB, PostgreSQL, and SQL Lite, but it doesn't work with DB2, Oracle, or SQL Server. Those RDBMSes use other proprietary commands instead of `LIMIT`.

## 4.3 Commenting data

Throughout this chapter, we've been discussing ways to sort and skip rows in your queries. I've provided some explanation of each query in terms of its purpose and considerations for executions, but if someone else read only the SQL we've used, would they understand why the queries were written the way they were? Probably not, which is why now is a good time to talk about comments. *Comments* allow you to include text in your query that isn't considered for execution. Typically, this text includes some kind of note to indicate the query author's intentions, as well as their identity and the date when the query was written or modified. Essentially, a comment can be any kind of information you want to include above and beyond the SQL you wrote.

Why would you want to use comments? RDBMSes aren't the only entities that are going to read your query; people will read it as well. These people could be colleagues who use the script, or the person who replaces you after you use your ever-expanding knowledge of SQL to secure a better job. Your comments can be as simple as your name and the date when you made your SQL script or as detailed as line-by-line descriptions of what each bit of SQL is meant to accomplish.

The downside of not using comments is ambiguity. Other people may look at your SQL and need to spend hours trying to figure out your intentions. Worse, you may look at some SQL you wrote weeks, months, or even years ago and be confused by what your former self wrote.

Writing descriptive, helpful comments is the mark of any well-respected SQL developer. Other people will have greater appreciation for your work because the extra seconds or minutes you spend writing clear comments will save them (and your future self) hours of confusion.

You have a few ways to write comments. First, you can *comment out* a particular line by using two consecutive hyphens:

```sql
-- This query returns three random rows
SELECT
    TitleName,
    PublicationDate
FROM title
LIMIT 3;
```

The use of two hyphens allows you to comment a single line of code up to the next carriage return. This type of comment is called an *inline comment*. In MySQL, you can also achieve an inline comment with the number sign (#):

```sql
# This query returns the three rows with the most recent PublicationDate
SELECT
    TitleName,
    PublicationDate
FROM title
ORDER BY PublicationDate DESC
LIMIT 3;
```

This type of comment isn't as common, so be aware that another RDBMS may not recognize it as a comment.

A third way to use comments is to surround your comment with `/*` and `*/`, encompassing your comment between those symbols, which allows for a multiline comment. You can comment out more than one line, as in the following example:

```sql
/* This query returns 3 TitleNames
...with the most recent PublicationDate
...excluding the single most recent TitleName */
SELECT
    TitleName,
    PublicationDate
FROM title
ORDER BY PublicationDate DESC
LIMIT 3 OFFSET 1;
```

##### Tip

Because they have greater functionality, multiline comments (using `/*` and `*/`) are the preferred method for use in reusable code. They can be especially useful when you want to comment out entire sections of SQL. You may want to do this to indicate a section of your SQL that doesn't execute as intended or to indicate a previous version of a query that you may want to reference later.

You can put multiple-line comments around single-line comments as well. You might have made a one-line comment about a particular SQL statement but later decided to comment out the entire statement with multiple-line comments, replacing it with different SQL. You could indicate this in the following way:

```sql
/*
# This query returns 3 random titles, but it wasn't what we needed
SELECT
    TitleName,
    PublicationDate
FROM title
LIMIT 3;
*/
-- This is the updated query, now ordered by most recent PublicationDate
SELECT
    TitleName,
    PublicationDate
FROM title
ORDER BY PublicationDate DESC
LIMIT 3;
```

Comments can be invaluable when you write a query, save it, and then come back to it weeks, months, or even years later to review it. I've been writing SQL for a few decades, and there have been innumerable times when I had to review the comments of an old query to determine the goal of that query. There have also been plenty of times when I looked at a query someone else wrote that had no comments, which led to many hours of trying to figure out what the writer intended the query to do.

Do yourself a favor: start developing the habit of carefully commenting any SQL you write, no matter how simple. Commenting takes only a few seconds, but as I mentioned, it could save you or someone else who reviews your code much more time. For this reason, all the SQL in this book's supplemental scripts has been commented to help you understand the purpose of each query.

## 4.4 Lab

1.  You need a list of all authors, but you need that list to be in alphabetical order. Write a query to return the FirstName and LastName of all authors, sorted by LastName and then FirstName.

2.  You need to write a query that returns all columns in the title table for only the highest-priced title. What does that query look like?

3.  Suppose that you have the following SQL statement, which gets all the carriage returns stripped out by the application executing it, and that this query ends up on a single line. What will the result of this query be?

```sql
-- Retrieve the book titles
SELECT TitleName
FROM title;
```

## 4.5 Lab answers

1.  The answer is

```sql
SELECT
    FirstName,
    LastName
FROM author
ORDER BY
    LastName,
    FirstName;
```

2.  The answer is

```sql
SELECT
    TitleID,
    TitleName,
    Price,
    Advance,
    Royalty,
    PublicationDate
FROM title
ORDER By
    Price DESC
LIMIT 1;
```

Alternatively, you could use `SELECT` `*` instead of listing all the column names.

3.  Because the entire query is now on a single line preceded by two hyphens, the entire query executes as a comment. Although it won't result in an error, it also won't return the results that the query intended. This situation is one of several reasons why I advise you to use `/*` and `*/` for comments in reusable code; both the beginning and end of the commented line or lines are clearly marked.

