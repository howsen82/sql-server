# 9 Using Different Kinds of Joins

Joining tables is an essential skill for writing SQL queries, but so far, we've tried only one kind of join. To be fair, that type of join is the most common, but as you'll see in this chapter, in plenty of scenarios, that kind of join won't help you produce the results you need.

You might be asked to produce a list of all orders for a given year, for example, and show whether they used a particular discount code. Or you might be asked to find the names of all customers who didn't place an order in a year. Or you might be asked to find a list of all customers in a particular city or state and show which ones placed orders and which did not. You can't accomplish these queries using the join type from chapter 8, so you're going to learn how to use different joins in SQL to fulfill all the preceding requests.

## 9.1 Inner joins

First, let's talk a little bit more about the `JOIN` keyword we used in chapter 8. This join is a shorthand version of the keyword `INNER` `JOIN`, which is a particular type of join. Because it joins only values in both tables that meet the conditions of the join, the results set excludes any rows that don't meet the conditions.

For much of this chapter, we'll use two tables: the orderheader table and a new table named promotion. This new table contains promotion codes that can be used for discounted prices on titles that are ordered. The primary key for the promotion table is PromotionID, and it's referenced by a similarly named PromotionID column in the orderheader table.

The reason we'll use these tables is that there are rows in each table that don't relate to the other table. That is, some rows in the promotion table represent promotions that were never used in any order, and some rows in the orderheader table represent orders that were placed without a promotion code. Also, the relationship between these tables is one-to-many: any promotion code can be used for more than one order, but every order can use only one promotion code.

Also, I should note that there are 12 rows in the promotion table and 50 rows in the orderheader table. I'll refer to the number of rows in these tables throughout the chapter.

##### Try it now

Use `SELECT` `*` `FROM` `promotion` and `SELECT` `*` `FROM` `orderheader` to see how many rows are returned. Check the message in the Output panel to confirm the number of rows in each table. If you prefer, you could count the rows returned yourself, but that's more time-consuming, as well as open to the possibility of human error.

Let's start with the following query, which finds the order ID and promotion code of any order that used a promotion code. Figure 9.1 shows a portion of the results of this query:

```sql
SELECT
    oh.OrderID,
    p.PromotionCode
FROM orderheader oh
JOIN promotion p
    ON oh.PromotionID = p.PromotionID;
```

##### Figure 9.1 A portion (8 of the 20 rows) of the results showing the OrderID and PromotionCode of all orders that used a promotion code

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH09_F01_Iannucci.png)

Looking at the Output window, we see the message `20` `row(s)` `returned`. You might notice that the values for PromotionCode are not 20 unique promotion codes, but the values for OrderID are 20 unique values. Again, this is because of the one-to-many relationship between the tables; each promotion code may be used for multiple orders.

If we want to be more verbose, we could write the same query by describing our join explicitly as an `INNER JOIN`, returning the same 20 rows:

```sql
SELECT
    oh.OrderID,
    p.PromotionCode
FROM orderheader oh
INNER JOIN promotion p
    ON oh.PromotionID = p.PromotionID;
```

##### Tip

If you're using only inner joins in a particular query, it's acceptable to write `JOIN` instead of `INNER` `JOIN`. But if a query will include other joins, such as those you're about to learn, you should specify `INNER` `JOIN` for clarity and readability.

A common way to show how the values in these tables relate to one another logically is a Venn diagram. Consider two intersecting circles, with each circle including the data of a single table. The intersecting parts of the circles represent the common values between the two tables, and the nonintersecting parts represent the values that are unique to each table.

Figure 9.2 is a Venn diagram of the inner join between the promotion and orderheader tables. We'll look at several Venn diagrams throughout this chapter to help visualize the data included in the different types of joins. The colored part of figure 9.2 represents the data that is returned in our result set, and the noncolored parts represent the data that is omitted from our results.

##### Figure 9.2 A Venn diagram of the inner join used in the preceding query

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH09_F02_Iannucci.png)

The data returned by our inner join includes only the common values that meet the condition of our join, where the PromotionID values in each table match. As I mentioned, this option isn't our only one for joining tables. We can also join the tables to include all the values of one table, even if the rows have no related values in the other table. We do this by using `OUTER` `JOIN` keywords.

## 9.2 Outer joins

The syntax used for outer joins is similar to that used for inner joins, in that you use the `JOIN` keyword to specify the tables you're joining and the `ON` keyword to identify the condition of the columns used in the relationship between the tables. If additional conditions for the join are necessary, they use the `AND` keyword.

One big difference between inner and outer joins is that there are different types of outer joins, so you need to state in your SQL which type of outer join you're using. Let's start with the `LEFT` `OUTER` `JOIN`.

### 9.2.1 Left outer joins

The use of the word *left* in `LEFT` `OUTER` `JOIN` indicates that we want all rows returned from the left table in our join, regardless of whether the rows match. If this keyword seems confusing, think of it as also meaning that we want all the rows from the *first* table noted in our join.

Suppose that we want to see a list of all order IDs, and if they used a promotion code, we want to see which promotion code was used. We'd use the same query as before but change our `INNER` `JOIN` to a `LEFT` `OUTER` `JOIN`:

```sql
SELECT
    oh.OrderID,
    p.PromotionCode
FROM orderheader oh
LEFT OUTER JOIN promotion p
    ON oh.PromotionID = p.PromotionID;
```

In this query, the left table is orderheader because it's the first table mentioned. In our formatted query, orderheader appears above promotion, not to the left. But if this query was contained on one unformatted line with no carriage returns, orderheader would be to the left of promotion in the query. We represent this kind of left outer join with a diagram like the one in figure 9.3.

##### Figure 9.3 A Venn diagram of the left outer join

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH09_F03_Iannucci.png)

The message in the Output panel lets us know that this query returns 50 rows, which is to be expected because we learned earlier that the orderheader table contains 50 rows. These 50 rows are a lot to take in, so let's add a filter to limit the results to the first 8 orders placed, as shown in figure 9.4:

```sql
SELECT
    oh.OrderID,
    p.PromotionCode
FROM orderheader oh
LEFT OUTER JOIN promotion p
    ON oh.PromotionID = p.PromotionID
WHERE oh.OrderID <= 1008;
```

##### Figure 9.4 The results show the OrderID and any PromotionCode used for the first eight orders. A PromotionCode was used only for OrderIDs 1006, 1007, and 1008.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH09_F04_Iannucci.png)

The result set includes a row for every row in the orderheader table that meets our condition of being less than or equal to OrderID 1008, regardless of whether we have a PromotionID value to join to the promotions table. For those orders that don't have a promotion code, the results show a null value in the PromotionCode column.

Although we've done a lot of work with filtering so far, we need to be careful when adding filtering conditions to outer joins. If we add a filtering condition for a specific value in the right table in the `WHERE` clause of our query with a `LEFT` `OUTER` `JOIN`, we effectively turn our `LEFT` `OUTER` `JOIN` into an `INNER` `JOIN`. Filtering on specific values in the right table would eliminate any rows from our result set that might have null values in the right table. We can demonstrate this situation with a similar query that filters on a particular value for the PromotionCode, with the results shown in figure 9.5:

```sql
SELECT
    oh.OrderID,
    p.PromotionCode
FROM orderheader oh
LEFT OUTER JOIN promotion p
    ON oh.PromotionID = p.PromotionID
WHERE p.PromotionCode = '2OFF2015';
```

##### Figure 9.5 The results of the left outer join with a filter on the right table (promotion), which reduces our result set to one that would be the same if the join were an inner join

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH09_F05_Iannucci.png)

Now the results include only three rows for orders that used PromotionCode 2OFF2015, not a row for every OrderID in orderheader, as we'd expect from a left outer join. Because the condition in the `WHERE` clause was required of the values in the right table, which in this query is promotion, that condition is applied to the results from both tables. Because of this filtering condition on the right table in the `WHERE` clause, we could change the `LEFT` `OUTER` `JOIN` in our query to `INNER JOIN` and get the same results.

##### Try it now

Execute the preceding query for a different PromotionCode, such as 2OFF2016. Execute it once with a `LEFT` `OUTER` `JOIN` and again with an `INNER` `JOIN` to see that the results are the same.

If we truly wanted to view a list of all orders and see whether they used a specific PromotionCode (such as 2OFF2015) instead of any PromotionCode, we could still do this. We just need to move the filtering from the `WHERE` clause to the join condition, like this:

```sql
SELECT
    oh.OrderID,
    p.PromotionCode
FROM orderheader oh
LEFT OUTER JOIN promotion p
    ON oh.PromotionID = p.PromotionID
    AND p.PromotionCode = '2OFF2015';
```

This query returns 50 rows—one for each OrderID—because we're not filtering on the left table. The value of the PromotionCode column in the result set is either 2OFF2015 or NULL.

Please make a note of how this filtering in the join condition works. On many occasions, you'll need to use an outer join while filtering on specific values in two or more tables. If you filter in the `WHERE` clause for a table joined with an outer join, you may inadvertently create an inner join.

### 9.2.2 Right outer joins

Just as the `LEFT` `OUTER` `JOIN` returns all rows in the left/first table regardless of whether they match the right/second table, the `RIGHT` `OUTER` `JOIN` does the opposite. It returns all rows in the *right* table whether or not they match rows in the left table with the join condition.

Let's use a right join to show all promotional codes regardless of their use, with a portion of the results shown in figure 9.6:

```sql
SELECT
    p.PromotionCode,
    oh.OrderID
FROM orderheader oh
RIGHT OUTER JOIN promotion p
    ON oh.PromotionID = p.PromotionID;
```

##### Figure 9.6 A portion (8 of the 23 rows) of the results showing all PromotionCodes and OrderIDs if they were used in a promotion

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH09_F06_Iannucci.png)

We can visually depict the results of our right join with the diagram shown in figure 9.7.

The preceding query returns 23 rows, which is more than the 12 rows in the promotion table. Look closely, and you'll see that many of the rows include duplicate values for PromotionCode because a code can be used for more than one order. Because of the duplicate use of certain PromotionCodes, we've matched many orders to some of the PromotionCodes.

##### Figure 9.7 A Venn diagram of the right outer join

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH09_F07_Iannucci.png)

Scroll through the results, and you'll also see that a few of the PromotionCode values have NULL for OrderID. Those PromotionCodes weren't used in a corresponding order, so they didn't match any order. We have them in our result set anyway because all rows in the promotion table, which is the right table in our query, will have at least one row in our result set from the right outer join.

### 9.2.3 Using outer joins to find rows without matching values

Just as we can use a left or right join to return all rows in a table regardless of whether they match, we can use either kind of outer join to find all the rows that don't match. We do this by saying explicitly that we want to find rows in the matching table with the filter condition `IS` `NULL`.

As an example, we can write a query to show only the PromotionCodes that were not used for any order, as shown in figure 9.8, by adding the filter `WHERE` `oh.PromotionID` `IS` `NULL` to the preceding query:

```sql
SELECT
    p.PromotionCode,
    oh.OrderID
FROM orderheader oh
RIGHT OUTER JOIN promotion p
    ON oh.PromotionID = p.PromotionID
WHERE oh.PromotionID IS NULL;
```

##### Figure 9.8 The results for all PromotionCodes that don't have a corresponding OrderID, meaning that the PromotionCode was never used

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH09_F08_Iannucci.png)

Although previously, I noted that placing a filtering condition on the joined query will effectively turn an outer join into an inner join, checking for null values is the exception. Remember that checking for a null value isn't checking for equality between two values, but querying for the presence of null values. This kind of query is common; you'll often have to find some value that exists in one table but not in another one. We can represent this concept with a diagram like the one in figure 9.9.

##### Figure 9.9 A Venn diagram of the right outer join that excludes rows from the left table

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH09_F09_Iannucci.png)

### 9.2.4 Interchanging left and right joins

If you try to execute the preceding query in SQLite, it won't work because that relational database management system (RDBMS) doesn't support the `RIGHT` `OUTER` `JOIN` command. This won't be a problem for most queries, however. You could simply rewrite the join as a `LEFT` `OUTER` `JOIN`:

```sql
SELECT
    p.PromotionCode,
    oh.OrderID
FROM promotion p
LEFT OUTER JOIN orderheader oh
    ON p.PromotionID = oh.PromotionID
WHERE oh.PromotionID IS NULL;
```

This query produces the same results as those shown in figure 9.8 because all we've done is swap the order of two tables in the `FROM` clause and change the join from `RIGHT` `OUTER` `JOIN` to `LEFT` `OUTER` `JOIN`. The diagram in figure 9.10 shows what we did.

##### Figure 9.10 A Venn diagram of the left outer join that excludes rows from the right table

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH09_F10_Iannucci.png)

##### Tip

As much as possible, try to use only left or only right outer joins in any query, not both. You'll need to include both types of outer joins in very few cases, and using only one type of outer join makes your query easier for other people to understand. Also, some RDBMSes don't support right outer joins. For this reason, we'll prefer left outer joins throughout the remainder of this book.

One last note about left and right joins: they don't need to be quite so verbose. Just as `INNER` `JOIN` can be shortened to `JOIN`, `LEFT` `OUTER` `JOIN` and `RIGHT` `OUTER` `JOIN` can be shortened to `LEFT` `JOIN` and `RIGHT` `JOIN`, respectively. The preceding query can be written without the `OUTER` keyword, which may improve readability:

```sql
SELECT
    p.PromotionCode,
 oh.OrderID
FROM promotion p
LEFT JOIN orderheader oh
    ON p.PromotionID = oh.PromotionID
WHERE oh.PromotionID IS NULL;
```

##### Note

Depending on your RDBMS, you may be able to use a `FULL OUTER JOIN` or `FULL` `JOIN`. This rarely used type of join returns the combined results of a `LEFT` `JOIN` and `RIGHT` `JOIN` of two tables. We won't be writing any queries with `FULL` `OUTER` `JOIN` because MySQL, Maria DB, and SQLite don't support this type of join, but we'll learn another way to produce this kind of result set in chapter 10.

### 9.2.5 The USING keyword

There are two other ways to write inner, left, right, and outer joins, but they're less common. The first way is to use the `USING` keyword, which replaces the `ON` keyword in the join and doesn't require us to specify the table names or aliases. We *can't* specify the table names or aliases because the `USING` keyword requires the names of the columns used in the relationship to be the same in both tables. We could rewrite the preceding query with `USING` to get the results shown in figure 9.8 like this:

```sql
SELECT
    p.PromotionCode,
    oh.OrderID
FROM promotion p
LEFT JOIN orderheader oh
    USING (PromotionID)
WHERE oh.PromotionID IS NULL;
```

The requirement that the column names be identical for a join is usually the biggest deterrent to using . . . well, `USING` because you'll encounter many databases with related tables that don't share column names. The `USING` command isn't used frequently, and many other SQL programmers aren't aware of it or its correct use. We're looking at it here only in case you notice it in someone else's SQL queries.

### 9.2.6 Natural joins

The second rare way to write inner, left, right, and outer joins is to use a natural join. With a *natural join,* you don't mention the column names involved in the relationship; seemingly by magic, they join columns of the same name from two tables. We do this by adding the `NATURAL` keyword while omitting `ON` or `USING`. Let's rewrite the preceding query one more time, this time with a natural join:

```sql
SELECT
    p.PromotionCode,
    oh.OrderID
FROM promotion p
NATURAL LEFT JOIN orderheader oh
WHERE oh.PromotionID IS NULL;
```

Although they further reduce the amount of SQL you have to write, I highly recommend that you avoid using natural joins. For starters, they don't identify the columns used in the relationship between the tables, so whoever reads your SQL will have no idea how the promotion and orderheader tables are related.

A bigger problem involves similarly named columns. Although our promotion and orderheader tables share only a single easily identifiable column with the same name, in real-world scenarios, many tables contain columns like CreateDate or ModifiedDate to track changes in the values of any rows. Although these columns are often similarly named in a given database, they aren't created to relate data. Using a natural join with tables that contain these commonly named columns would automatically join the data in those columns, which wouldn't produce the expected results.

##### Warning

Natural joins are not supported in SQL Server.

## 9.3 Cross joins

The last kind of join we'll look at is another unusual one: the cross join. What makes this type of join unusual is that unlike all the other joins we've discussed, it isn't used to find rows with specific values. Rather, a *cross join* finds all possible combinations of rows by matching every row from one table to every row in another.

The cross join is also known as a *Cartesian join* because the results of the cross join reflect the mathematical operation known as a Cartesian product, which describes this result set of all possible paired values from two sets of data. If we want to use a cross join to show all possible combinations of PromotionCodes from the promotion table and OrderIDs from the orderheader table, we could write a query like this:

```sql
SELECT
    p.PromotionCode,
    oh.OrderID
FROM promotion p
CROSS JOIN orderheader oh;
```

The results of this query reflect all possible combinations of matching values from the two tables, so it should be no surprise that the result set for this query is 600 rows. We have 12 rows in the promotion table and 50 rows in the orderheader table, so a little multiplication (12 × 50) confirms that 600 rows are to be expected in the results.

Although a cross join isn't helpful for finding particular rows, it's useful when you need to generate a full list of all possible outcomes. You might need to produce a grid for all sizes and colors of a particular product, for example. It can also be beneficial for quickly generating a lot of test data, such as a list of customers or orders that are much larger than what your current data contains.

##### Warning

Although I previously noted that you don't need to specify an inner join with the word `INNER` in MySQL, if you join tables using only the word `JOIN` and omit a join condition, the results will reflect a cross join, not an inner join as you intended. Other RDBMSes may require `ON` when using `INNER` joins.

The cross join is the last of many types of joins we've examined in this chapter. It may seem discouraging that several of the shorter ways to write SQL joins are not recommended, but in chapter 10, we'll look at other ways to join data—more efficient methods that are highly accepted and sometimes more efficient for the RDBMS. For now, let's practice what you've learned.

## 9.4 Lab

At the beginning of this chapter, I discussed some scenarios that you might encounter. Let's start by using what you've learned to write queries to produce those results. Try to use a left join in each of the first three exercises:

1.  Write a query that shows the OrderID and OrderDate of all orders from 2019, as well as the PromotionCode, if one was used.

2.  Write a query to show the first and last names of all customers who didn't place an order in 2020.

3.  Write a query to show the first and last names of customers as well as the OrderID and OrderDate for any orders placed in 2021 by customers in California (where the value for State in the customer table is CA).

4.  This exercise wasn't mentioned at the beginning of the chapter, but write a query using a cross join to generate a list of all possible customer first names from the customer table and last names from the author table.

## 9.5 Lab answers

1.  The answer is

```sql
SELECT
    oh.OrderID,
    p.PromotionCode
FROM orderheader oh
LEFT JOIN promotion p
    ON oh.PromotionID = p.PromotionID
WHERE oh.OrderDate >= '2019-01-01'
    AND oh.OrderDate < '2020-01-01';
```

2.  The answer is

```sql
SELECT
    c.FirstName,
    c.LastName
FROM customer c
LEFT JOIN orderheader oh
    ON c.CustomerID = oh.CustomerID
    AND oh.OrderDate >= '2021-01-01'
    AND oh.OrderDate < '2022-01-01'
WHERE oh.CustomerID IS  NULL;
```

3.  The answer is

```sql
SELECT
    c.FirstName,
    c.LastName,
    oh.OrderID,
    oh.OrderDate
FROM customer c
LEFT JOIN orderheader oh
    ON c.CustomerID = oh.CustomerID
    AND oh.OrderDate >= '2021-01-01'
    AND oh.OrderDate < '2022-01-01'
WHERE c.State = 'CA';
```

4.  The answer is

```sql
SELECT
    c.FirstName,
    a.LastName
FROM customer c
CROSS JOIN author a;
```

