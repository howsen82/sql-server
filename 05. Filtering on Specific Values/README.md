# 5 Filtering on Specific Values

So far, you've been writing mostly queries that return an entire set of data, but as you write more purposeful SQL using larger sets of data, you'll find that you need only a subset of the data instead of all the rows. You did work a bit in chapter 4 to reduce the number of rows returned using `LIMIT` and `OFFSET`, but those commands aren't helpful for finding specific rows.

You may want only a report of sales for the past month, a list of orders with pending status, or a list of customers in New Hampshire, for example. All these scenarios have *conditions* for specific data being returned, and we apply those conditions using filtering. *Filtering* means taking the broader results of your data set and applying one or more conditions to restrict the data being returned. To do this, you primarily use a different clause: the `WHERE` clause.

It's highly likely that most of the SQL you'll write in your career will include a `WHERE` clause because there's a nearly infinite number of ways you may need to find data that meets specific criteria. The `WHERE` clause is incredibly powerful, with so many ways to filter data that it will take a few chapters to review them. Let's get started!

## 5.1 Filtering on a single condition

The most basic methods for filtering data are relatively intuitive and easy to learn. The main variations involve the type of data you're querying. As you may have noticed, there are different types of data—such as names, numbers, and dates—and each type has slightly different rules for filtering. We'll look at them all in this section, starting with filtering by using a condition with a numeric value.

### 5.1.1 Filtering on numeric values

Suppose that we want to know the TitleName of any Titles for which the Advance for the author was $10,000. Let's start by declaring a sentence: "I would like the title name of the title where the advance is 10,000 dollars."

Notice that grammatically, we not only use the word *where* for our filtering but also place our filtering condition toward the end of the sentence. In SQL, we're going to do the same thing, and we could write a query for this request like this:

```sql
SELECT TitleName
FROM title
WHERE Advance = 10000.00;
```

Let's take a closer look at that `WHERE` clause and examine the rules that govern it:

* As in the preceding SQL query, the `WHERE` clause comes after the `FROM` clause, as it naturally would in English.
* Notice that we use an equal sign (`=`) instead of the word *is.* The use of the equal sign indicates equality, which means that our filtering condition is looking for values equal to a specific value. In this case, the use of the equal sign makes a lot of sense.
* Note that we have no dollar sign or comma in `10000.00`. Although we use commas to make numeric values more readable and use dollar signs to indicate currency, this data is typically stored as a number, and the computer that runs your relational database management system (RDBMS) doesn't care about a specific currency type or the readability of the numbers. Using dollar signs and commas in this case would be problematic.

##### Warning

When you start filtering for large numeric values, such as orders above $1 million, it can be tempting to put commas in the numeric values to make the data more readable. After all, it can be easy to mistakenly type 1000000 as 100000 or 10000000. Unfortunately, including commas in numeric values will cause syntax errors for your query.

Although currency types and commas can't be used with numeric values, decimals can often be added or removed without changing the results of the data. This is possible because numeric values can be equal, even if they don't have the same precision. *Precision* refers to the mathematical specificity of a value, and how precise the data is depends on how that data is stored and how you're querying it.

As an example, 1.00 is more precise than 1. We can increment 1.00 up to 1.01, but the next incremental value after 1 is 2. For this reason, the latter is less precise. For querying, though, even though 1.00 is more precise, it's mathematically the same as 1.

In the case of the Advance value we just used to filter, let's take a quick look at the value of the data shown with the following query (results shown in figure 5.1):

```sql
SELECT
    TitleName,
    Advance
FROM title
WHERE Advance = 10000.00;
```

##### Figure 5.1 Only one row meets the filter criteria for an Advance value of 10000.00.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH05_F01_Iannucci.png)

In terms of precision, $10,000.00 is more precise than $10,000, but numerically, the numbers are the same value. For this reason, we can write a query without the decimal values used to represent cents and still get the results shown in figure 5.1:

```sql
SELECT TitleName, Advance
FROM title
WHERE Advance = 10000;
```

##### Try it now

Use the previous two queries to test the `WHERE` clause; see how the results are the same. Also try using an even more precise value in your filter condition, such as `10000.0000`. All the results should be the same.

### 5.1.2 Filtering on string values

So far, our queries have filtered on numeric conditions, but filtering on non-numeric conditions is a bit different. Instead of looking for a TitleName for a specific Advance, let's reverse that approach and query for the Advance of a specific TitleName (results shown in figure 5.2):

```sql
SELECT Advance
FROM title
WHERE TitleName = 'Anne of Fact Tables';
```

##### Figure 5.2 The result of a query for the Advance from the title table where the TitleName is Anne of Fact Tables

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH05_F02_Iannucci.png)

Now the filter condition isn't a numeric value but a group of words. To the RDBMS, this group of words is a set of characters known as a *string value*, and any time we filter on a string, we need to place single quotes around the value. If we don't, our query will result in a syntax error.

##### Warning

Not all single quotes work. You must use the single quote on the same keyboard key as the double quotes for this query to work. If you use the tick mark next to the 1/! key on most keyboards, you'll get a syntax error. Also, if you copy and paste code from a document other than a SQL script, you may get incorrectly formatted single quotes like the ones in figure 5.3.

##### Figure 5.3 Incorrect single quotes, copied from a Microsoft Word document. Workbench is letting you know this with the square (red onscreen) with an X to the left of the line.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH05_F03_Iannucci.png)

The value of the string used in our `WHERE` clause must be an exact match, as the slightest variation will prevent us from getting the intended results. If we forgot the letter *s* in `Tables` as in the following query, we won't get any results:

```sql
SELECT Advance
FROM title
WHERE TitleName = 'Anne of Fact Table';
```

##### Note

Forgetting a character such as that last *s* may seem like a clear mistake, but subtle mistakes involving characters that don't seem like characters—such as extra spaces, tabs, and carriage returns—can lead to incorrect results. Although the RDBMS ignores the use of those characters in the format of SQL queries, it treats them as extra characters in your string values.

### 5.1.3 Filtering on date values

*Date values* have their own considerations in filtering because they're used as a kind of hybrid of numeric and string values. Like string values, date and time values must also be enclosed in single quotes. If you want to find the TitleName with a PublicationDate of March 14, 2020, for example, use the following SQL:

```sql
SELECT
    TitleName,
    PublicationDate
FROM title
WHERE PublicationDate = '2020-03-14 00:00:00';
```

The default format is year-month-day and then hours:minutes:seconds. The use of single quotes here may seem obvious because this value contains non-numeric characters, such as dashes and colons. But what if I told you that the RDBMS you're using is storing date and time values as numeric values? It's true. Storing those values this way is more efficient than storing them as a string of characters.

What this means for us and our use of SQL is that the rules of precision that apply to numeric values also apply to date and time values. Consider that the `00:00:00` in the filtering value represents hours:minutes:seconds—specifically, the exact second of midnight at the start of a day. If no time is provided, these zeroes are included in the default value for a date, as they are in the results of the most recent query (figure 5.4).

##### Figure 5.4 The TitleName and PublicationDate for a title published March 14, 2020. The time values are the equivalent of midnight at the start of the day.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH05_F04_Iannucci.png)

Now, given that we know 10,000.00 is numerically the same as 10,000, we can conclude that 2020-03-14 00:00:00 is also the same as 2020-03-14. Because the value of the data in the table has all zeroes for hours, minutes, and seconds, we can be confident that we'll get the results shown in figure 5.4 by writing the query without consideration of time, like this:

```sql
SELECT
    TitleName,
    PublicationDate
FROM title
WHERE PublicationDate = '2020-03-14';
```

##### Tip

As your SQL skills progress, it will be helpful to be aware of the kind of data included in the tables you're going to query. It's easy to see that all the values for publication date in the title table have no precision for hours, minutes, or seconds and that we can safely query that data without including that level of time precision. But if a single value has so much as a second of precision, it would be better to write queries that account for the time as well.

## 5.2 Filtering on multiple conditions

So far, you've filtered on only a single condition, but in real-world queries, you often need to filter on multiple conditions. Imagine that you have to find a customer whose first name is Jeff and whose last name is Iannucci, or an Order that's number 1001 and an Item that's Product X. For queries like these, the `WHERE` clause allows filtering on multiple conditions using an intuitive method.

### 5.2.1 Filtering that requires all conditions

Suppose that we want to query the title table for Title Names for which the Advance is $5,000 and the Royalty is 15%. We would verbally declare our request like this: "I would like the TitleNames from title where the Advance is 5,000 dollars and the Royalty is 15 percent."

The use of the word `AND` to add to our filtering criteria is intuitively included in our SQL (results shown in figure 5.5):

```sql
SELECT
    TitleName,
    Advance,
    Royalty
FROM title
WHERE Advance = 5000
    AND Royalty = 15;
```

##### Figure 5.5 Although multiple rows in the title table have a Price value of 5000.00, adding a second filter condition for a Royalty of 15.00 (%) reduces the result set to one row.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH05_F05_Iannucci.png)

In this context, the keyword `AND` is considered an *operator*, which means that it's a keyword that performs a specific operation in SQL. In the case of `AND`, the operation is to join multiple filter conditions within the `WHERE` clause. `AND` is the first of several operators we'll discuss.

The use of `AND` allows us to add as many filter conditions as we need for a given query, even beyond the example in the preceding query. Even though the result of the previous query is only one row, theoretically, we could refine the filter with additional criteria by putting another `AND` filter condition in the `WHERE` clause:

```sql
SELECT
    TitleName,
    Advance,
    Royalty,
    PublicationDate
FROM title
WHERE Advance = 5000
    AND Royalty = 15
    AND PublicationDate = '2015-04-30';
```

##### Try it now

Use the preceding two queries to test the `WHERE` clause; see how the results change with each filter. Try filtering first with a condition where the Royalty is 12; then add another filter condition where Advance is 6000. The rows in the result set should reduce from two rows in your first query to one row in the second.

Although there's no specific limit to how many filter conditions you can include in your `WHERE` clause, understand that for rows to be included in your result set, they must meet every one of the filter conditions. Failure to meet any single condition will exclude the rows from the results.

### 5.2.2 Filtering that requires any one of many conditions

Although the `AND` operator allows us to filter on multiple conditions that all rows must meet, sometimes we want to apply multiple filter conditions and get results that simply meet *one or more* of the conditions.

What if we want to find any book that has either an Advance of $5,000 or a Royalty of 15%? We could verbally declare the request like this: "I would like the TitleNames from title where the Advance is 5000 dollars or the Royalty is 15 percent." The word *or* has replaced *and* in our sentence, and it will do the same in our SQL:

```sql
SELECT
    TitleName,
    Advance,
    Royalty
FROM title
WHERE Advance = 5000
    OR Royalty = 15;
```

If this query is executed, we'll get very different results from the previous query, which used the `AND` operator instead of the `OR` operator. Now we have six rows returned instead of one (figure 5.6) because rows in the results need to meet either condition to be included—not both.

##### Figure 5.6 The results for any rows in the title table that have either an Advance of 5000.00 or a Royalty of 15.00 (%)

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH05_F06_Iannucci.png)

We can use the `OR` operator the same way as the `AND` operator, in that we can add as many filter conditions as our query requires. Note that for each `AND` operator we add, a result set could get smaller as the conditions become more restrictive, whereas each `OR` operator could result in larger result sets as the conditions become more inclusive.

We can increase the result set from six rows to seven by adding an `OR` operator for Price and the Price column to the result set like this because any given row included in the results in figure 5.7 needs to meet only one of three filter conditions to be included:

```sql
SELECT
    TitleName,
    Advance,
    Royalty,
    Price
FROM title
WHERE Advance = 5000
    OR Royalty = 15
    OR Price = 9.95;
```

##### Figure 5.7 The seven rows that can match any of the three conditions of an Advance of 5000.00, a Royalty of 15.00 (%), or a Price of 9.95

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH05_F07_Iannucci.png)

We can go on and on adding filter conditions with multiple `OR` statements. Each time, we'll get as many rows as the time before—or more rows as the filter conditions become more inclusive.

### 5.2.3 Controlling the order of multiple filters

In some situations, we need to filter in a way that requires using both `AND` and `OR` in the `WHERE` clause. But we need to be very careful about how we do this.

Suppose that we need to find a list of TitleNames with a Price of $9.95 and either a PublicationDate of February 6, 2016 or an Advance of $5,000. To find this data, we might try writing a query like this:

```sql
SELECT
    TitleName,
    Price,
    PublicationDate,
    Advance
FROM title
WHERE Price = 9.95
    AND PublicationDate = '2016-02-06'
    OR Advance = 5000;
```

This query looks correct, but if we execute it, we'll find that the results don't match our intended logic. The RDBMS reads this query differently from what we intended, as shown in figure 5.8. This happens because the RDBMS prioritizes `AND` conditions over `OR` conditions, regardless of our intentions.

##### Figure 5.8 The rows returned by the query don't match our intended filter conditions for TitleNames with a Price of 9.95 and a PublicationDate of 2016-02-06 or an Advance of 5000.00.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH05_F08_Iannucci.png)

Let's list the logic of our intended filtering conditions, either of which needed to be met:

* A Price of 9.95 and a PublicationDate of 2016-02-06
* A Price of 9.95 and an Advance of 5000.00

Because the RDBMS places a higher priority on `AND` conditions than on `OR` conditions, it determines the written filtering conditions differently from what we intended. To the RDBMS, our SQL is requesting rows that meet either of these conditions:

* A Price of 9.95 and a PublicationDate of 2016-02-06
* An Advance of 5000.00

The results in figure 5.8 show the only title that met the first condition (The Join Luck Club) and three others that met the second condition.

Getting this logic correct can be one of the most confusing problems for SQL beginners, but ironically, the solution is simple: all you need to do is use parentheses to explicitly prioritize your logic over the default SQL logic. Anything inside the parentheses is evaluated before anything outside the parentheses. To get the results we intended, we'd use parentheses in the preceding query like this:

```sql
SELECT
    TitleName,
    Price,
    PublicationDate,
    Advance
FROM title
WHERE Price = 9.95
    AND (PublicationDate = '2016-02-06'
    OR Advance = 5000);
```

Now this query will evaluate the values as intended, evaluating the `OR` condition before the `AND` condition. Executing this query returns a result set like the one shown in figure 5.9.

##### Figure 5.9 With the parenthetical notation, the query returns data that has a Price of 9.95 and either a PublicationDate of 2016-02-06 or an Advance of 5000.00.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH05_F09_Iannucci.png)

##### Tip

Whenever you write any SQL that uses both `AND` and `OR` operators in your `WHERE` clause, always use parentheses to explicitly control the evaluation. This approach not only helps the RDBMS figure out your intentions but also reduces guesswork for anyone else who will be evaluating your code.

### 5.2.4 Filtering and using ORDER BY

As you've gone through this chapter, you may have wondered what happened to the `ORDER` `BY` clause from chapter 4 and how it fits in with the `WHERE` clause in this chapter. When you use both `WHERE` and `ORDER` `BY` clauses in your SQL, you need to write the `ORDER BY` clause *after* the `WHERE` clause.

We can use a query from earlier in the chapter that returned four rows. Figure 5.7 shows the rows returned in no particular order, but if we wanted the results to be sorted by TitleName, we could easily add an `ORDER` `BY` clause like this (results shown in figure 5.10):

```sql
SELECT
    TitleName,
    Price,
    PublicationDate,
    Advance
FROM title
WHERE Price = 9.95
    AND PublicationDate = '2016-02-06'
    OR Advance = 5000
ORDER BY TitleName;
```

##### Figure 5.10 The four rows that match any of three conditions—a Price of 9.95, a PublicationDate of 2016-02-06, or an Advance of 5000.00—with the results sorted alphabetically by TitleName

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH05_F10_Iannucci.png)

We've covered a lot of examples of basic filtering today. It's time to use our new skills!

## 5.3 Lab

1.  If we don't place single quotes around a non-numeric string value in our filter condition, we know that we'll get a syntax error. Try placing single quotes around a numeric value like Price in a filter condition and executing a query. What happens?

2.  Why does the following query return no results?

```sql
SELECT
    TitleName,
    Price
FROM Title
WHERE TitleName = 'Anne of Fact Tables ';
```

3.  What will the result of the following query be?

```sql
SELECT TitleName
FROM Title
ORDER BY TitleName ASC
WHERE Price = 9.95;
```

4.  Write a query using the author table that returns rows with either a PaymentMethod of `Check` and a FirstName of `Jorge` or a PaymentMethod of `Check` and a LastName of `Miller`. Include the columns FirstName, LastName, and PaymentMethod in your result set.

## 5.4 Lab answers

1.  The query executes as expected, although behind the scenes, the RDBMS has made the data match. It had to convert either the value in quotes to a number or all the values in the Price column to a string, which could negatively affect the duration of the query if we were using a larger set of data. For this reason, avoid putting single quotes around numeric values.

2.  The query returns no values because there is an extra space after the word `Tables` in the `WHERE` clause. For a string used in a filter to be correct, it needs to be exactly like the values in the table, including unseen characters such as spaces, tabs, and carriage returns.

3.  The query will result in a syntax error because the `ORDER` `BY` can't come before the `WHERE` clause.

4.  This query is similar to the one in section 5.2.3. Because it uses `AND` and `OR` and parentheses, the query should return two rows and should look something like this:

```sql
SELECT
    FirstName,
    LastName,
    PaymentMethod
FROM author
WHERE PaymentMethod = 'Check'
AND (FirstName = 'Jorge'
    OR LastName = 'Miller');
```

