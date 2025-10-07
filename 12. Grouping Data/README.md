# 12 Grouping Data

If you're accustomed to working with spreadsheets, rather than relational data in a database, the past three chapters may have been a bit challenging for you. After all, in spreadsheets you often work with a single set of data instead of multiple sets. If the concepts in those chapters are new to you, take heart; this chapter covers concepts that should be very familiar to most spreadsheet users.

One useful aspect of spreadsheets is that they allow us to do mathematical calculations on a range of data quickly. If we want to find the total of all values in a column, for example, we can click the AutoSum button, which places the desired sum amount in a particular cell. If we highlight that cell, we see that the spreadsheet used the word `SUM` with the defined range of cells. `SUM` represents a *function*, which is a command that performs a predefined calculation.

Although we have no button in the SQL language to calculate totals automatically, we do have functions like `SUM` to help us perform mathematical calculations. Moreover, in a relational database, we have much more flexibility in the way we perform these calculations than we have in spreadsheets.

## 12.1 Aggregate functions

Throughout the rest of this book, we'll be discussing different kinds of functions in SQL. A *function* is a keyword that makes it easy to perform calculations or other actions. SQL has many functions for calculating all sorts of values for converting dates, formatting data, and doing much more.

This chapter focuses on the main *aggregate functions*, which are functions that perform a calculation over a range of data in a column. When you need to perform basic calculations, these aggregate functions are indispensable.

### 12.1.1 The SUM function

The most basic function we can use is `SUM`, which returns the sum total of all values in a column of data. If we want to know the total number of titles ordered, for example, we could sum the Quantity column in the orderitem table. We could declare this intention verbally by using the word *sum* to describe what we want: "I would like the sum of the quantity of titles in the orderitem table." This declaration isn't far off from our SQL, which looks like this (results shown in figure 12.1):

```sql
SELECT SUM(Quantity)
FROM orderitem;
```

##### Figure 12.1 The quantity of all orders is shown in a column with no alias. The default column name is the calculation.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH12_F01_Iannucci.png)

Note a few things before we go any further. First, we need to use parentheses to identify what column we're choosing for the `sum`. If we don't use the parentheses, the query will result in a syntax error.

Also, if you execute this query, you'll notice that the column uses your calculation as the name of the column, which may not be helpful. When you use aggregate functions, you usually want to specify a name for the column by using an alias, which helps you identify the meaning of the returned values. If you want to execute this query again, you should modify it to use an alias that reflects the aggregate calculation:

```sql
SELECT SUM(Quantity) AS TotalQuantity
FROM orderitem;
```

##### WARNING

Keep in mind that the `SUM` function is intended to be used only with numeric data values. If you try to use this function with date or character values, the result may not be meaningful.

##### Tip

When you use an alias for an output column, avoid using the name of the table column as the alias. In addition to possibly confusing anyone who might read your output, some relational database management systems (RDBMSes) don't allow this action.

### 12.1.2 The COUNT function

Although `SUM` calculates the sum total of all values, if we want to know the quantity of values that exist in a column, we need to use a different function. The `COUNT` function counts the number of rows in a column. This statement seems relatively obvious, but note one important feature of aggregate functions: by default, they exclude null values.

Let's look at the orderheader table to see how many rows it contains. We can do this easily by selecting the `COUNT` of all the OrderIDs, for which every row has a value. Figure 12.2 shows the result of this query:

```sql
SELECT COUNT(OrderID) AS TotalOrders
FROM orderheader;
```

##### Figure 12.2 A total of 50 rows in the orderheader table have a value for OrderID, which is all the rows in the table.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH12_F02_Iannucci.png)

The results indicate that we have 50 rows in the orderheader table, which is correct. But if we try to count the number of PromotionCodes used by selecting the `COUNT` of the PromotionID column, we'll get a different result (shown in figure 12.3):

```sql
SELECT COUNT(PromotionID) AS TotalOrdersWithPromotionCode
FROM orderheader;
```

##### Figure 12.3 Only 20 rows in the orderheader table have a value for PromotionID.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH12_F03_Iannucci.png)

The results now show only 20 rows, which means that only 20 of the 50 rows in the orderheader table have a value for PromotionID. The other 30 rows have a null value for PromotionID.

The `COUNT` function also has a unique, widely used feature: you can use it to return the number of rows in a table without specifying a column. If you don't know the names of any columns in the orderheader table, you could easily find them by using the asterisk (`*`), as you learned when selecting all columns in chapter 3:

```sql
SELECT COUNT(*) AS TotalOrders
FROM orderheader;
```

The results of this query will be the same as those shown in figure 12.2, which is noteworthy because even if there are null values in any of the columns, selecting `COUNT(*)` always returns the total number of rows in a table.

##### Warning

`SELECT` `COUNT(*)` is a useful method for quickly determining the number of rows in most tables, but be careful when using it with tables that have millions or billions (or more) rows. This kind of query can use excessive computer resources and create delays for other queries.

### 12.1.3 The MIN function

The `MIN` function returns the minimum, or lowest, non-null value for a column. If we want to find the least expensive item from all orders, as shown in figure 12.4, we can use a query like this:

```sql
SELECT MIN(ItemPrice) AS MinimumItemPrice
FROM orderitem;
```

##### Figure 12.4 The minimum price of any item in the orderitem table is $4.95.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH12_F04_Iannucci.png)

### 12.1.4 The MAX function

`MIN` has a commonly used partner function: the `MAX` function. Whereas the `MIN` function returns the lowest value for a row, the `MAX` function returns the maximum, or highest, value. Let's change the function used in the preceding query to find the highest price for any item, as shown in figure 12.5:

```sql
SELECT MAX(ItemPrice) AS MaximumItemPrice
FROM orderitem;
```

##### Figure 12.5 The maximum price of any item in the orderitem table is $12.95.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH12_F05_Iannucci.png)

Although the `SUM` function can be used only with numeric data, in MySQL and many other RDBMSes, you can use the `MIN` and `MAX` functions with non-numeric data. When these functions are used with non-numeric data, they return the first or last value, respectively, as though the data were sorted on that column in ascending order. Although using `MIN` and `MAX` with date values is rarely problematic, the warnings for string values about collation in earlier chapters apply here as well: sometimes lower- and uppercase letters, as well as nonalphabetic characters, are ranked differently by different collations.

##### Try it now

Write a short query to select the `MIN` value for the FirstName column of the author table.

### 12.1.5 The AVG function

The last aggregate function we'll use in this chapter is `AVG`, will computes the average of all non-null values in a column. If we want to find the average price of the titles in the title table, as shown in figure 12.6, we could use the `AVG` function like this:

```sql
SELECT AVG(Price) AS AveragePrice
FROM title;
```

##### Figure 12.6 The average price of all titles in the title table

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH12_F06_Iannucci.png)

It looks as though the average price of the titles in our database is about $9.70. You surely noticed a lot of extra zeros in the result; those zeros appear because the `AVG` function is attempting to calculate the average value to a higher level of precision. This situation isn't a problem because the value is accurate; chapter 14 discusses how to modify the precision of the result if necessary.

##### WARNING

Like the `SUM` function, the `AVG` function should be used only with numeric values.

### 12.1.6 Filtering and aggregating combined values

We can also use a filter with our aggregate functions. Suppose that we want to determine the average price of all titles published after January 1, 2019. Let's try our verbal declaration: "I would like the average price of all titles with a publication date greater than January 1, 2019." This declaration converts easily to the following SQL query:

```sql
SELECT AVG(Price) AS AveragePrice
FROM title
WHERE PublicationDate > '2019-01-01';
```

We can even combine our functions in the same query. If we want to know the dates of the first and last orders in the orderheader table, for example, we can use the `MIN` and `MAX` values because they work with date values:

```sql
SELECT
    MIN(OrderDate) AS FirstOrder,
    MAX(OrderDate) AS LastOrder
FROM orderheader;
```

Something else we can do with aggregate queries is use a mathematical calculation inside the parentheses. We need to do this in queries that require the values from more than one column, such as determining the total dollar value of all items sold. If we conclude that we can determine the total dollar value for any row of items by multiplying Quantity by ItemPrice, we can use that calculation with a `SUM` function to determine the total sales value for all the rows in our orderitem table, with the result shown in figure 12.7:

```sql
SELECT SUM(Quantity * ItemPrice) AS TotalOrderValue
FROM orderitem;
```

##### Figure 12.7 The total value of all orders in the orderitem table is $573.50.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH12_F07_Iannucci.png)

As easy as it is to determine total overall sales, until now we've been limited to evaluating values at table level or basing evaluations on filtered data from a table. What if we want to know the `SUM` of each order, the quantity of items sold for each PromotionCode, or the number of titles sold by each author?

## 12.2 Aggregating data with GROUP BY

If we want to analyze data at a deeper level, we need a new set of keywords: `GROUP BY`. `GROUP` `BY` isn't just a couple of keywords but also a new clause that allows us to divide one set of data into groups on which we can perform our aggregations. That explanation may seem theoretical, so the following section provides a practical example.

### 12.2.1 GROUP BY requirements

In the preceding query, we selected the total dollar value for all orders. If we want to find the total dollar value for each individual order, we would divide our data into groups of values for each order and then perform the same calculation as previously. We divide our values by grouping them, in this case by OrderID. Given the formula we used before, a verbal declaration might look something like this: "I would like the sum of the quantity multiplied by the item price of all the ordered items, and I want to group the sum by order ID."

Here is how this declaration looks as a query. We'll add an `ORDER` `BY` to sort the data by OrderID for readability (result shown in figure 12.8):

```sql
SELECT OrderID, SUM(Quantity * ItemPrice) AS OrderTotal
FROM orderitem
GROUP BY OrderID
ORDER BY OrderID;
```

##### Figure 12.8 Eight of the 50 rows returned show the calculated OrderTotal for all orders.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH12_F08_Iannucci.png)

This query is similar to the last one except that now we're grouping our data into logical sets of values for each OrderID and then calculating `SUM(Quantity` `*` `ItempPrice)` for each set. We do this by adding first `GROUP` `BY` `OrderID` and then `ORDERID` to our `SELECT` clause.

##### Note

When you use `GROUP` `BY`, every column in `SELECT` must be included in the `GROUP` `BY` clause or must have an aggregate calculation. If a column in `SELECT` doesn't meet either requirement, you'll get a syntax error.

You may have noticed that the `GROUP` `BY` clause is used after the `FROM` clause and before the `ORDER` `BY` clause. `GROUP` `BY` needs to be after the `WHERE` clause as well, if we have one. Knowing this, if we want to limit our orders to those placed after January 1, 2019, we must join to our orderheader table to add this filter, and we probably should alias the column names as well:

```sql
SELECT
    oi.OrderID,
    SUM(oi.Quantity * oi.ItemPrice) AS OrderTotal
FROM orderitem oi
INNER JOIN orderheader oh
    ON oi.OrderID = oh.OrderID
WHERE oh.OrderDate > '2019-01-01'
GROUP BY oi.OrderID
ORDER BY oi.OrderID;
```

Adding this filter on OrderDate reduces the result set from 50 rows to 21, but keep in mind that we are still performing the calculation on each data set based on OrderID for those 21 rows.

### 12.2.2 GROUP BY and null values

The `GROUP BY` clause has another useful feature: it allows us to perform aggregations on columns with null values. You may remember from section 12.1.2 that we used the `COUNT` function to find promotions used in orders, and the null values were excluded. When we use the `GROUP BY` clause, we can account for null values too.

Suppose that we want to account for those 30 orders that were placed without a promotion code, as shown in figure 12.9. Without a promotion code, these orders are represented by a null value in the PromotionID column of the orderheader table. Because these PromotionID values are null, they were excluded from our query with `COUNT` in section 12.1.2. But we can use `GROUP BY` to group the orders logically by `PromotionID` because `GROUP` `BY` includes null values:

```sql
SELECT PromotionID, COUNT(*) AS RowCount
FROM orderheader
GROUP BY PromotionID
ORDER BY PromotionID;
```

##### Figure 12.9 The PromotionID values in the orderheader table. This result includes null values because `GROUP` `BY` doesn't exclude nulls.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH12_F09_Iannucci.png)

In this query, we're not filtering the results. Filtering can be a bit tricky when we use `GROUP` `BY` because the aggregations can't be filtered in the `WHERE` clause. If we want to filter the results of our aggregations, we have to introduce another clause.

## 12.3 Filtering with HAVING

The `HAVING` clause is the partner clause to `GROUP` `BY`, in that it's where we filter on the aggregations we're calculating. It's used similarly to the `WHERE` clause. The main difference is that the `WHERE` clause filters rows, whereas the `HAVING` clause filters groups of aggregated values. The good news is that we can apply any of the filtering methods we've learned in the `HAVING` clause if necessary.

Let's work through an example. Suppose that we want to find the PromotionCodes used in orders and to see which ones were used at least three times. We could start by writing a query to find all PromotionCodes used, joining the orderheader table to promotion on PromotionID, grouping the values by PromotionCode, and then counting the times a PromotionID was used in the orderheader table. We'll alias the tables and order the results for readability:

```sql
SELECT
    p.PromotionCode,
    COUNT(oh.PromotionID) AS OrdersWithPromotionCode
FROM orderheader oh
INNER JOIN promotion p
    ON oh.PromotionID = p.PromotionID
GROUP BY p.PromotionCode
ORDER BY p.PromotionCode;
```

##### Note

If you've executed all the queries in this chapter so far, you may wonder what happened to the null values in the results of the preceding query. Well, those null values for PromotionID exist in the orderheader table, but they don't exist in the promotion table, And even if they did, an `INNER` `JOIN` would exclude them, so the results include only matching values from both tables.

Now that we have our basic query to view which PromotionCodes were used and how often they were included in an order, we can add the `HAVING` clause to filter on codes used three or more times, with the results shown in figure 12.10:

```sql
SELECT
    p.PromotionCode,
    COUNT(oh.PromotionID) AS OrdersWithPromotionCode
FROM orderheader oh
INNER JOIN promotion p
    ON oh.PromotionID = p.PromotionID
GROUP BY p.PromotionCode
HAVING COUNT(oh.PromotionID) >= 3
ORDER BY p.PromotionCode;
```

##### Figure 12.10 The only PromotionCodes that were used at least three times

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH12_F10_Iannucci.png)

Now that we've added our `HAVING` clause, we've filtered our results down to three rows. If we want to, we can use the alias of our aggregation (`OrdersWithPromotionCode`) in the `HAVING` clause like this:

```sql
SELECT
    p.PromotionCode AS PromoCode,
    COUNT(oh.PromotionID) AS OrdersWithPromotionCode
FROM orderheader oh
INNER JOIN promotion p
    ON oh.PromotionID = p.PromotionID
GROUP BY p.PromotionCode
HAVING OrdersWithPromotionCode >= 3
ORDER BY p.PromotionCode;
```

The query should return the results shown in figure 12.10. This outcome may seem contradictory because in earlier chapters, I noted that you can't use a column alias in the `WHERE` clause. The next section is probably a good time to talk about something that's important in SQL and the queries we write: the logical order in which the RDBMS reads our queries.

## 12.4 Logical query processing

Up to this point, you've learned about several clauses in SQL statements and the order in which you must write them. At a simplified level, SQL clauses are ordered like this:

1. `SELECT`
2. `FROM` (including `JOIN`s)
3. `WHERE` (including `AND`s and `OR`s)
4. `GROUP` `BY`
5. `HAVING`
6. `ORDER` `BY`

This order, however, isn't the one in which the MySQL RDBMS reads your queries. It reads them in this order:

1. `FROM` (including `JOIN`s)
2. `WHERE` (including `AND`s and `OR`s)
3. `SELECT`
4. `GROUP` `BY`
5. `HAVING`
6. `ORDER` `BY`

This order in which the RDBMS reads your queries is known as *logical query processing*, which defines the logical order in which the RDBMS processes your query. This concept is important to understand because it will not only help you troubleshoot your queries when they return unexpected or incorrect results but also help you determine when you can use table and column aliases.

The order of logical query processing may seem strange, but it's optimal for the RDBMS to follow in processing your query:

1. Evaluate the data in the tables your query will use in the `FROM` clause.
2. Filter the data to reduce the result set in the `WHERE` clause.
3. Gather the columns to be returned in the `SELECT` clause.
4. Group those columns in the `GROUP BY` clause for aggregation.
5. Filter the aggregations in the `HAVING` clause.
6. Sort the results in the `ORDER` `BY` clause.

Now you see why table aliases can be used throughout the query: they're logically established in the earliest processing of our query in the `FROM` clause. You also see why you can use column aliases established in the `SELECT` clause in the `HAVING` and `ORDER` `BY` clauses but not in the `WHERE` clause.

##### Warning

Although MySQL performs logical query processing as described in this section, other RDBMSes may follow a different order by logically processing the `SELECT` clause after `GROUP` `BY` and `HAVING`. For this reason, you shouldn't get into the habit of using column aliases in your `HAVING` clause.

## 12.5 The DISTINCT keyword

We have one more keyword to cover in this chapter: `DISTINCT`. This helpful keyword is used frequently, but it's a bit misunderstood.

We can use the `DISTINCT` keyword in a `SELECT` clause to avoid having repeating values. If we want to see the names of all titles ever ordered, as shown in figure 12.11, we could write a query like this using `DISTINCT`:

```sql
SELECT DISTINCT t.TitleName
FROM title t
INNER JOIN orderitem oi
    ON t.TitleID = oi.TitleID
ORDER BY t.TitleName;
```

##### Figure 12.11 The distinct title names included in orders in the orderitem table

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH12_F11_Iannucci.png)

Although the table contains 50 orders, some of which were placed for more than one title, the query returns only one row for each title. `DISTINCT` can be very useful for determining the range of values in any table quickly, and I'm sure you'll see it used often in other people's queries. So why is it covered in a chapter about aggregation? When you use `SELECT` `DISTINCT`, your RDBMS is doing an aggregation to return your distinct values, and that aggregation is extra work. By using `DISTINCT` in the preceding query, we're essentially asking the RDBMS to process this query:

```sql
SELECT t.TitleName
FROM title t
INNER JOIN orderitem oi
    ON t.TitleID = oi.TitleID
GROUP BY t.TitleName
ORDER BY t.TitleName;
```

##### Try it now

Run the two preceding queries with `DISTINCT` and `GROUP` `BY`, respectively, and notice that they both produce the results shown in figure 12.11.

Now that you're aware of what `DISTINCT` does, try to limit its use in your SQL, especially with large data sets. Aggregating data isn't problematic when you're querying the small tables in this book's example database, but it can be difficult when you're querying larger tables elsewhere.

##### TIP

One of the most common misuses of `DISTINCT` is to eliminate duplicates from query results when multiple data sets are joined. If you're tempted to use `DISTINCT` to remove duplicates from the results of a query with multiple joins, take a second look at the join conditions to make sure you're joining on the correct columns. Incorrect joins often cause unwanted duplicate rows in a result set.

## 12.6 Lab

1.  In this chapter, I noted that you can't use certain functions with certain data types. To better understand this limitation, select the `SUM` of the OrderDate in the orderheader, and see what the result is.

2.  Why won't this query work?

```sql
SELECT
    p.PromotionCode AS PromoCode,
    COUNT(oh.PromotionID) AS OrdersWithPromotionCode
FROM orderheader oh
INNER JOIN promotion p
    ON oh.PromotionID = p.PromotionID
WHERE PromoCode = '2OFF2015'
GROUP BY p.PromotionCode
HAVING OrdersWithPromotionCode >= 3
ORDER BY p.PromotionCode;
```

3.  Write a query to count the number of rows in the author table.

4.  Write a query to select the minimum and maximum values of the publication dates from the titles table.

5.  In this chapter, you determined the total dollar value of all orders by using the equation `Quantity` `*` `ItemPrice` on the orderitem table. Write a query using `GROUP` `BY` to determine the average total dollar value for each individual order. (Hint: you may need to use a subquery.)

## 12.7 Lab answers

1.  You may have missed it, but in chapter 5, I noted that date and time values are stored as numeric values that your RDBMS can interpret as dates and times. Although it's practically useless, the value you see here is the RDBMS trying to make sense of using a `SUM` of date values.

2.  The query fails because a column alias is used in the `WHERE` clause, and the logical query processing order is to evaluate the `WHERE` clause before the `SELECT` clause. For this reason, the RDBMS doesn't know what PromoCode is when it evaluates the `WHERE` clause because the alias is determined in the `SELECT` clause that is processed later in the query.

3.  To count the number of rows in a table, you can use `COUNT(*)`:

```sql
SELECT COUNT(*)
FROM author;
```

4.  To select the minimum and maximum publication dates, you can use the `MIN` and `MAX` functions:

```sql
SELECT
    MIN(PublicationDate) AS FirstPublication,
    MAX(PublicationDate) AS LastPublication
FROM title;
```

5.  You have a few ways to do this. The first way is to group the total value of all orders as we did in section 12.2.1 and then select an average value of those order totals, like this:

```sql
SELECT AVG(OrderTotals.OrderTotal)
FROM (
    SELECT OrderID, SUM(Quantity * ItemPrice) AS OrderTotal
    FROM orderitem
    GROUP BY OrderID
    ) OrderTotals;
```

