# 10 Combining Queries with Set Operators

In the past few chapters, we examined ways to join tables based on the way they relate to one another. Every query we've written has had a single `SELECT` statement. But this chapter will show how to write a query with multiple `SELECT` statements and combine the results into a single set of data.

This technique can be useful when we need to evaluate results that require different conditions, such as querying values in different tables with no key to join them. Although we've seen that null values are excluded from results when we use joins, we'll see how to use SQL to include null values if those values exist in two data sets and we want to include them in our results.

## 10.1 Using set operators

We've written a lot of queries that start with `SELECT`, and each resulted in a single result set. That's what `SELECT` queries do: produce a set of results. More specifically, they produce a set of rows that meet the various conditions of our queries.

At times, though, we want to combine or evaluate two or more result sets, and to do this, we need to use special keywords known as *set operators*. Though SQL doesn't have many set operators, all of them use the same syntax to evaluate two result sets:

```sql
SELECT <some column>, <another column>
FROM <some table>
WHERE <some condition>
<set operator>
SELECT <some column>, <another column>
FROM <some table>
WHERE <another condition>;
```

Even though we're using two different `SELECT` statements in our query, the set operator allows us to evaluate the results into a single result set. The most common evaluation is to combine them, but as we'll see later in this chapter, we can do more. First, though, we must adhere to a few rules for using set operators:

* *The number of columns needs to match.* This rule is the most obvious one because we know from our introduction to tables that every row in a table must have the same number of columns. Our result set using a set operator is no different. Attempting to evaluate queries with a different number of columns will result in an error.
* *The data type of each column needs to match.* We haven't talked much about data types yet, but we've seen that there are different data types for numbers, characters, and dates. If we attempt to combine different data types in a result set, we'll receive an error message.
* *The names of columns in the first query are used in the result set.* This rule means that we can evaluate columns with different names, but their ordinal positions must be the same in each `SELECT` statement. If we're using column aliases, only those used in the first `SELECT` statement apply to the results. We can add column aliases to any `SELECT` statement other than the first one in our queries without causing an error, but remember that the relational database management system (RDBMS) will ignore those aliases, which won't affect the result set. Understanding this rule is also important because of the last rule.
* *An `ORDER` `BY` clause can appear only in the final `SELECT` statement.* An `ORDER` `BY` is the last evaluation in any query, and as such, it's allowed only after the last `SELECT` statement. If we try to sort the results in any other `SELECT` statement, we'll receive an error message.

## 10.2 UNION

The most common set operator is `UNION`, which allows us to combine the results of two or more `SELECT` statements into a single result set, removing any duplicates.

##### Tip

One of the most important things to remember about `UNION` is that it removes duplicate rows from your result set. Don't forget!

As an example, we can combine the names of all the people in our sqlnovel database into a single set of names. That is, we can select the first and last names from the customer and author tables into a single result set. Let's select the names from tables and order by last name and first name (results shown in figure 10.1):

```sql
SELECT FirstName, LastName
FROM customer
UNION
SELECT FirstName, LastName
FROM author
ORDER BY LastName, FirstName;
```

##### Figure 10.1 A portion (8 of the 31 rows) of the results of first and last names from the customer and author tables

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH10_F01_Iannucci.png)

The results of our two `SELECT` statements have been combined into a single result set and ordered as we directed in figure 10.1. But why do we use the word `UNION`? Well, if we wanted to verbally declare what we're requesting with our SQL, we'd say something like this: "I would like the first and last names from the customer table, and I would like to combine the results with the first and last names from the author table."

Although the word *combine* accurately describes what we're doing, as we'll see throughout this book, there are all sorts of ways to combine things in SQL. We can combine rows, columns, values, and entire result sets in different ways, so the word *combine* isn't specific enough to describe what we're requesting. Instead, we use the word *union* to describe combining two or more data sets into a single result.

In English, the noun *union* often describes the uniting in marriage of people from two different families into one new family, so it may be helpful to think of the `UNION` operator as marrying two different data sets into a single result set. Our verbal declaration becomes a bit more descriptive with the word *union:* "I would like the first and last names from the customer table, and I would like to union the results with the first and last names from the author table."

Although we don't inherently know whether any given row was selected from the customer table or the author table, we can verify the table of origin by adding a third column with literal values to indicate the table from which the row came. We'll add these literal values to both `SELECT` statements, but we need to add the column name only to the first `SELECT` statement, as shown in figure 10.2. As I noted earlier, the column names in the result set are chosen from the first `SELECT` statement:

```sql
SELECT FirstName, LastName, 'customer' TableName
FROM customer
UNION
SELECT FirstName, LastName, 'author'
FROM author
ORDER BY LastName, FirstName;
```

##### Figure 10.2 A portion (8 of the 31 rows) of the results of first and last names from the customer and author tables, as well as a third column indicating the table that the rows came from

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH10_F02_Iannucci.png)

The most common way to use a union is to combine various filtering conditions, especially ones that might be contradictory, in a single `SELECT` statement, such as values from different tables. Let's add some filtering conditions for LastName from the customer table and FirstName from the author table (results shown in figure 10.3):

```sql
SELECT FirstName, LastName, 'customer' TableName
FROM customer
WHERE LastName LIKE 'D%'
UNION
SELECT FirstName, LastName, 'author'
FROM author
WHERE FirstName LIKE 'C%'
ORDER BY LastName, FirstName;
```

##### Figure 10.3 The results for the full names of customers whose last name starts with *D* and authors whose first name starts with *C*, ordered by last name and first name

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH10_F03_Iannucci.png)

In figure 10.3, only five rows meet the filtering criteria, and we have at least one row from each table. Notice that there are duplicate values for first name (Chris) and last name (Daly). Recall from earlier in this chapter that duplicate rows in which all values match are removed from our results when we use `UNION`.

We can verify this fact by making a few changes in our query. Two rows in figure 10.3 have the first name Chris, which appears once in each table we're querying. Let's omit the LastName and TableName columns from our results because those extra columns create unique rows for the rows that have Chris as a value for FirstName. With these columns omitted, we should expect Chris to appear in only one row because duplicates will be removed when we use `UNION`.

In addition to omitting those columns, we'll change the `ORDER` `BY` from LastName to FirstName. When we're using a set operator like `UNION`, we can sort the results only by columns included in the `SELECT` statement. We've removed LastName from the `SELECT` statement, so if we left the SQL for ordering by LastName in our next query, we'd get an error on execution, indicating that LastName is an "unknown column." Here's our new query (results shown in figure 10.4):

```sql
SELECT FirstName
FROM customer
WHERE LastName LIKE 'D%'
UNION
SELECT FirstName
FROM author
WHERE FirstName LIKE 'C%'
ORDER BY FirstName;
```

##### Figure 10.4 The results of first names from the customer table whose last names start with *D*, combined with a `UNION` with the results of first names from the author table whose first names start with *C*. The two rows for Chris are represented by a single row because `UNION` removed the duplicate row from the result set.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH10_F04_Iannucci.png)

But what if we don't want the duplicate rows to be removed? What if instead, we want any duplicate rows to be *included* in a result set? For that scenario, let's look at the next set operator: `UNION` `ALL`.

## 10.3 UNION ALL

The `UNION` `ALL` set operator is very much like the `UNION` operator, with the main difference being that it doesn't remove duplicate rows. Instead, `UNION` `ALL` instructs the RDBMS to read all the data as requested by each `SELECT` statement and return the results as they were read. We can modify the set operator from the preceding query from `UNION` to `UNION` `ALL` to see this difference (results shown in figure 10.5):

```sql
SELECT FirstName
FROM customer
WHERE LastName LIKE 'D%'
UNION ALL
SELECT FirstName
FROM author
WHERE FirstName LIKE 'C%'
ORDER BY FirstName;
```

##### Figure 10.5 The results of first names from the customer table whose last names start with *D,* combined with a `UNION ALL` with the results of first names from the author table whose first names start with *C*

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH10_F05_Iannucci.png)

Both rows for Chris are included in the results in figure 10.5 because `UNION` `ALL` doesn't remove duplicate rows from the result set. Because `UNION` `ALL` doesn't remove duplicates, the filtering conditions it uses can function similarly to the filtering in a `WHERE` clause. There are no differences between the results of these two queries, for example:

```sql
SELECT LastName
FROM customer
WHERE LastName = 'Daly'
UNION ALL
SELECT LastName
FROM customer
WHERE LastName = 'Dixon'
ORDER BY LastName;

SELECT LastName
FROM customer
WHERE LastName = 'Daly'
    OR LastName = 'Dixon'
ORDER BY LastName;
```

##### Try it now

Execute the two preceding queries to verify that both return the same results.

Both queries return a result set with three rows—two for Daly and one for Dixon—although they do so in different ways. The first query executes two `SELECT` statements against the customer table and then combines the results, with each query searching for rows that meet a single condition. The second query instead executes a single `SELECT`, which searches for rows that meet multiple conditions.

##### Tip

As you progress in your knowledge of SQL, it's always good to know different ways to produce the same results because you may encounter situations in which one technique performs better than others. Depending on factors that are covered in later chapters, a query with a `UNION` `ALL` may produce results much faster than a similar one with an `OR`, even though the `UNION` `ALL` is executing two SQL statements to find the same results. For now, remember to always consider different ways to find results if your query performs worse than expected.

One other difference between `UNION` and `UNION` `ALL` is that queries with `UNION` are generally slower than those with `UNION` `ALL` because the RDBMS has to do more work to remove the duplicate rows. If the result set from a `UNION` `ALL` is very large and full of duplicates, however, `UNION` queries could be faster because there would be less data in the result set to send over the network. Keep these facts in mind when you write SQL statements that query hundreds of gigabytes of data or more when using either `UNION` or `UNION` `ALL`.

Last, we can use `UNION` `ALL` to achieve the same results as a `FULL` `OUTER` `JOIN`, which (as noted in chapter 9) is not a supported type of join in MySQL, MariaDB, and SQLite.

## 10.4 Emulating FULL OUTER JOIN in MySQL

As noted at the end of chapter 9, a `FULL` `OUTER` `JOIN` returns not only the rows that match between two tables but also the unmatched rows from both tables. Figure 10.6 shows how we would represent these results in a Venn diagram if we could write such a query by searching for promotion codes shared by the promotion and orderheader tables.

##### Figure 10.6 A Venn diagram of the values included in a query of the orderheader table with a `FULL` `OUTER JOIN` of the promotion table

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH10_F06_Iannucci.png)

Here's what the SQL would look like. This type of query won't execute in MySQL, but it will execute in another RDBMS that allows `FULL` `OUTER` `JOIN`:

```sql
SELECT
    p.PromotionCode,
    oh.OrderID
FROM orderheader oh
FULL OUTER JOIN promotion p
    ON oh.PromotionID = p.PromotionID
```

The results of a `FULL` `OUTER` `JOIN` are very similar to the results of executing a `LEFT` `JOIN` and `RIGHT` `JOIN` at the same time, but because we can't do this in MySQL, we can emulate it with the left and right joins combined with a `UNION ALL`. `UNION` `ALL` is preferred in this case because it doesn't remove duplicate rows, which may exist between the tables and would be returned with a `FULL` `OUTER` `JOIN`.

The one tricky thing to remember is that when we're emulating a `FULL` `OUTER` `JOIN` in this way, we need to modify one of the joins to exclude common values. If we don't, we will end up with duplicate rows because both left and right joins include the common values that an `INNER` `JOIN` would return.

The following query demonstrates how to emulate a `FULL` `OUTER` `JOIN` in MySQL using `UNION ALL`. We'll prevent the duplicate representation of matching rows by excluding them from our second `SELECT` by filtering on `WHERE` `oh.PromotionID` `IS` `NULL`, which we learned about in chapter 9:

```sql
SELECT
    p.PromotionCode,
    oh.OrderID
FROM orderheader oh
LEFT JOIN promotion p
    ON oh.PromotionID = p.PromotionID
UNION ALL
SELECT
    p.PromotionCode,
    oh.OrderID
FROM orderheader oh
RIGHT JOIN promotion p
    ON oh.PromotionID = p.PromotionID
WHERE oh.PromotionID IS NULL;
```

This query returns 53 rows, which include

* Rows that match PromotionID values in both tables
* Rows in orderheader that don't contain PromotionID values
* Rows in promotion that contain PromotionID values that weren't used in the orderheader table

It may be helpful to use Venn diagrams to show specifically what each of the two `SELECT` statements in our query is doing. The first query finds the first two of our three sets of rows noted earlier, which are rows that match PromotionID values in both tables and rows in orderheader that don't contain PromotionID values. The diagram in figure 10.7 represents these values.

The second query finds the last item, which are rows in promotion that contain PromotionID values that weren't used in the orderheader table. The diagram in figure 10.8 represents these values.

By using a `UNION` `ALL`, we effectively combined all these results into a single result, as shown in figure 10.6 earlier in this section.

`UNION` and `UNION` `ALL` are useful set operators. But nearly every RDBMS has two other set operators that you should know about: `INTERSECT` and `EXCEPT`.

##### Figure 10.7 A Venn diagram of the values included in a query of the orderheader table with a left outer join of the promotion table

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH10_F07_Iannucci.png)

##### Figure 10.8 A Venn diagram of the values included only in a query of the promotion table, with a right outer join of the orderheader table

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH10_F08_Iannucci.png)

##### Warning

MySQL didn't support `INTERSECT` and `EXCEPT` until version 8.0.31. If you're using an earlier version, you'll encounter errors when attempting to use these operators.

## 10.5 INTERSECT

Another set operator that may prove useful is `INTERSECT`, which can return results similar to those of a query with an `INNER` `JOIN`. There are two important differences between their results, however:

* Where `INNER` `JOIN` will return duplicate values, `INTERSECT` will not. This fact is similar to the differences in the results of `UNION ALL` and `UNION`.
* An `INNER` `JOIN` will never return null values because nothing can't equal nothing. Because `INTERSECT` is looking for common values between two data sets and not evaluating equality, the results of `INTERSECT` will also include any null values that match in the results of the two queries.

Let's review an inner join to demonstrate the differences. Here's how we can use an `INNER` `JOIN` to find PromotionID values in both the orderheader and promotion tables:

```sql
SELECT
    oh.PromotionID
FROM orderheader oh
INNER JOIN promotion p
    ON oh.PromotionID = p.PromotionID;
```

If we wrote this query for an RDBMS that supports `INTERSECT`, our query would look like this:

```sql
SELECT
    PromotionID
FROM orderheader
INTERSECT
SELECT
    PromotionID
FROM promotion;
```

Although we used only one column in this example, `INTERSECT` supports the use of multiple columns in your `SELECT` statements. Although the column names don't need to be the same, all queries must have the same number of columns, and the columns must be selected in the same order for `INTERSECT` to evaluate them.

## 10.6 EXCEPT

Another set operator frequently supported by other RDBMSes can return data in one set that is not included in a second set. That set operator is `EXCEPT`, which excludes results similar to a method we learned about in discussing left joins in chapter 9.

##### Note

Oracle doesn't support the `EXCEPT` operator, but it has a `MINUS` operator that is identical in function and use.

Suppose that we want to find all the PromotionID values in the promotion table that weren't used in any orders in the orderheader table, as represented by the diagram in figure 10.9. We already know that we can find them with a statement like this:

```sql
SELECT
    p.PromotionID
FROM promotion p
LEFT JOIN orderheader oh
    ON p.PromotionID = oh.PromotionID
WHERE oh.PromotionID IS NULL;
```

##### Figure 10.9 A Venn diagram of the values included only in a query of the promotion table, with a left outer join of the orderheader table

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH10_F09_Iannucci.png)

We can achieve a result set very similar to this use of a `LEFT` `JOIN` that filters on null matches in the joined table by using `EXCEPT`:

```sql
SELECT
    PromotionID
FROM promotion
EXCEPT
SELECT
    PromotionID
FROM orderheader;
```

As with `INTERSECT`, the results of `EXCEPT` have two significant differences from those of `LEFT` `JOIN`:

* A `LEFT` `JOIN` will return duplicate values, but `EXCEPT` won't.
* A `LEFT` `JOIN` will never return null values because nothing can't equal nothing. Because `EXCEPT` is looking for common values between two data sets, not evaluating equality, the results of `EXCEPT` will also include any null values that exist in the results of the first query and not the second.

Although `INTERSECT` and `EXCEPT` are used far less frequently than `INNER` `JOIN` and `LEFT JOIN`, keep them in mind if you're using an RDBMS that supports these keywords and you need to have null values returned in your result set.

That's plenty of information on how to use set operators. In chapter 11, we'll examine other ways to join tables and other data sets by using logical operators.

## 10.7 Lab

1.  In this chapter, we learned that the column names in queries with `UNION` and `UNION` `ALL` come from the first `SELECT` statement. What do you think will happen with a query like this one that has no column name for the last column in the first query? Try it to find out:

```sql
SELECT FirstName, LastName, 'customer'
FROM customer
UNION
SELECT FirstName, LastName, 'author' TableName
FROM author
ORDER BY LastName, FirstName;
```

2.  Considering that there are rows in the customer table for customers Cora Daly and Kevin Daly, will the results of these two queries be the same? If not, what will the differences be?

```sql
SELECT LastName
FROM customer
WHERE FirstName = 'Cora'
OR FirstName = 'Kevin';

SELECT LastName
FROM customer
WHERE FirstName = 'Cora'
UNION
SELECT LastName
FROM customer
WHERE FirstName = 'Kevin';
```

3.  We've looked at the different behaviors of `UNION` and `UNION ALL`, but we haven't used them in the same query. It's inadvisable to use them in the same query, though, and depending on the RDBMS you're using, you may get an error message if you attempt to do so. Even if you don't, the results can be unpredictable. To demonstrate this situation, try executing the following two queries. Notice that they have different results. Why do you think the results are different?

```sql
SELECT LastName
FROM customer
WHERE FirstName = 'Cora'
UNION
SELECT LastName
FROM customer
WHERE FirstName = 'Kevin'
UNION ALL
SELECT LastName
FROM customer
WHERE LastName = 'Daly';

SELECT LastName
FROM customer
WHERE LastName = 'Daly'
UNION ALL
SELECT LastName
FROM customer
WHERE FirstName = 'Kevin'
UNION
SELECT LastName
FROM customer
WHERE FirstName = 'Cora';
```

## 10.8 Lab answers

1.  Because no column name is supplied for the last column in the first `SELECT` statement, the literal value `'customer'` is used for the final column name in the result set.

2.  The results won't be the same. The first query, which uses `OR` for filtering, returns two rows—one for each match. The second query, which uses `UNION`, returns only one row because `UNION` removes duplicates from the results.

3.  The first query returns three rows, and the second query returns one. At first glance, these results may be confusing because the second query simply reverses the order of the `SELECT` statements.

The answer lies in precedence, which indicates the order in which the `SELECT` statements are presented. The first `UNION` or `UNION` `ALL` is applied first, and the second one is applied next. Remember that `UNION` eliminates duplicates, and `UNION` `ALL` doesn't.

In the first query, each of the first two `SELECT` statements returns one row with the value `'Daly'`, and because the `UNION` eliminates duplicates, the result is one row—so far. But the third `SELECT` statement returns two rows, which, when evaluated with a `UNION` `ALL`, create a result set of three rows—one row from the first two `SELECT` statements and two rows from the last `SELECT` statement.

In the second query, we combine the first `SELECT` statement, which returns two rows, with the second `SELECT` statement, which returns one row, and we do this with a `UNION` `ALL`. If we executed only this part of the query, we'd get three rows returned. But then we combine another `SELECT` statement that returns one row with a `UNION`, which removes all the duplicates from our result set. That's why the second query returns only one row.

If this explanation is more than a little confusing, don't worry. As long as you avoid combining `UNION` and `UNION` `ALL` in the same query, you'll never have to worry about this headache.

