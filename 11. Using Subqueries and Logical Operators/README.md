# 11 Using Subqueries and Logical Operators

In chapter 10, we expanded the scope of our thinking a bit. We saw how to use SQL not only to query tables but also, with the help of set operators such as `UNION` or `INTERSECT`, to combine the results of two or more `SELECT` statements to form a single result set. In this chapter, we'll build on that knowledge by examining an important method of evaluating the results of multiple `SELECT` statements in the same query: the subquery.

*Subqueries* are simply queries nested into another query. We use subqueries when we can't achieve the desired results from a single `SELECT` statement, so instead of writing two or more queries, we combine them into a single query. Don't worry—this process isn't as complicated as it sounds.

By the end of this chapter, you'll see how subqueries allow you to evaluate the results of `SELECT` statements in ways beyond the capabilities of the set operators you learned about in preceding chapters. We have a lot of ways to use subqueries to discover, so let's get started.

## 11.1 A simple subquery

As I've noted throughout the book, a `SELECT` statement is a type of SQL query that returns a set of data known as a *result set*. So far, you've executed dozens of queries that produce result sets. You've queried one or more tables, producing result sets that look a lot like tables. By that, I mean the results have rows and columns, and the columns have names.

Let's take that scenario a step further. Because query results produce a result set similar to a table, we can evaluate the results of `SELECT` statements in many of the same ways that we evaluate the data in a table. This means we can join and filter these results as though they were tables, and the way to do this is to use subqueries. Rather than continue to speak theoretically, I'll give you an example.

Suppose that we want to find the order ID and order date for any orders placed after a particular order by a customer named Margaret Montoya. We're using this customer because they placed only one order. If we want to verbally declare this request, we might say the following: "I would like the order ID, customer ID, and order date from the customer table, but I want only the orders placed after the one order placed by the customer named Margaret Montoya."

This statement is the first time we've declared something in a compound sentence, with two separate clauses. With what we've learned so far, we could write two separate queries to get the desired results. The first query, to find the order placed by Margaret Montoya, might look like this:

```sql
SELECT
    oh.OrderID,
    oh.OrderDate
FROM orderheader oh
INNER JOIN customer c
    ON oh.CustomerID = c.CustomerID
WHERE c.FirstName = 'Margaret'
    AND c.LastName = 'Montoya';
```

The result of this query shows that Margaret Montoya placed only one order, on April 23, 2021. Now that we have that date, we could include it in a second query to find the results shown in figure 11.1. The query might look something like this:

```sql
SELECT
    OrderID,
    CustomerID,
    OrderDate
FROM orderheader
WHERE OrderDate > '2021-04-23';
```

##### Figure 11.1 The OrderID, CustomerID, and OrderDate of all orders placed after Margaret Montoya placed her only order on April 23, 2021

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH11_F01_Iannucci.png)

Writing two queries to find this result is cumbersome, which is why we'd replace the hardcoded date with a subquery. All we need to do is replace the hardcoded order date of `'2021-04-23'` with the first query, which is now a subquery, and surround the subquery with parentheses.

##### Warning

When filtering with a subquery, we can have only one column returned because SQL allows us to evaluate only one value or set of values at a time in the `WHERE` clause. If we select more than one column in the subquery used next, our query will result in an error.

Here's what a `SELECT` statement with a subquery that produces the same results as the two preceding queries might look like:

```sql
SELECT
    OrderID,
    CustomerID,
    OrderDate
FROM orderheader
WHERE OrderDate > (
    SELECT
        oh.OrderDate
    FROM orderheader oh
    INNER JOIN customer c
        ON oh.CustomerID = c.CustomerID
    WHERE c.FirstName = 'Margaret'
        AND c.LastName = 'Montoya'
    );
```

The results of this query are the same as the results shown in figure 11.1 because we've combined the logic of two queries into one.

The subquery has the containing parentheses on different lines above and below the subquery. Although you don't need to format your subqueries this way, this method of formatting has benefits. For one thing, the code is a bit easier to read than it would be if you placed the parentheses at the beginning and end of the subquery. Perhaps more important, it's easier to drag the cursor over the subquery, highlight only the subquery, and execute it alone. When you're writing your own SQL, this technique is helpful for verifying that the subquery produces the desired results.

##### Try it now

Execute the preceding subquery. Then highlight and execute only the lines of the subquery inside the parentheses to validate that the value returned is `2021-04-23`.

In this query, we used a comparative operator, `>`, with our subquery, but other comparison operators such as `=` and `<>` have the limitation of being able to compare only one value. To unlock the full potential of subqueries, we need to use an entirely different set of keywords, known as *logical operators*.

## 11.2 Logical operators and subqueries

Logical operators are a bit like comparison operators in that they test whether some condition is true, false, or unknown. You've already used a few of these operators: `IN`, `NOT` `IN`, `BETWEEN`, and `LIKE`. Some comparison operators are unable to evaluate a result set with more than one row, so let's look at an example of a subquery that returns more than one row.

OrderID 1034 includes more than one title, as we see in figure 11.2. If we need to write SQL to find what titles are included, we can write something like this:

```sql
SELECT
    t.TitleName
FROM title t
INNER JOIN orderitem oi
    ON oi.TitleID = t.TitleID
WHERE OrderID = 1034;
```

##### Figure 11.2 The four titles included in order 1034

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH11_F02_Iannucci.png)

Let's rewrite this query using a subquery by moving the part of the query that filters on OrderID into its own query, place that part in the `WHERE` clause as a subquery, and filter on where the TitleID from the title table is equal, using `=` to match the value of the results of our subquery:

```sql
SELECT
    t.TitleName
FROM title t
WHERE TitleID = (
    SELECT TitleID
    FROM orderitem
    WHERE OrderID = 1034
    );
```

This query doesn't work, though. If you try executing it, the Output panel displays the error message "Subquery returns more than 1 row." This is true. We know from the preceding query that this order includes four titles, which means that the subquery returns four rows. Our subquery can't be evaluated by `=` because that comparison operator is trying to determine whether every TitleID in the title table equals a single value. This query would work if there were only one title in the order, but there are four, and unfortunately, the `=` operator can't evaluate more than one value. For this scenario, our first logical operator, `ANY`, can help.

##### Try it now

Execute the preceding query, and notice the error in the Output panel.

### 11.2.1 The ANY and IN operators

The `ANY` logical operator evaluates a set of values to see whether any of them have equality to the values you're attempting to match—hence, the name `ANY`. You can think of `ANY` as being a helper for `=` (or any other comparison operator), allowing any value to be included in the subquery. To get the preceding query to work, simply add the `ANY` operator after `=` in the `WHERE` clause:

```sql
SELECT
    t.TitleName
FROM title t
WHERE TitleID = ANY (
    SELECT TitleID
    FROM orderitem
    WHERE OrderID = 1034
    );
```

The results of this query are the same as the results shown in figure 11.2. We can verbally declare what we're doing like this: "I would like the title name from the title table, and I would like the titles to match any of the titles from order 1034."

##### Note

Most relational database management systems (RDBMS), including MySQL, also support the `SOME` logical operator, which is identical to `ANY` in use and function. It's used much less frequently than `ANY`, however.

We can also get the same results by replacing `ANY` with a different logical operator: the `IN` keyword. We can make a similar verbal declaration of our intention: "I would like the title name from the title table, and I would like the titles to be in titles from order 1034." Our query would look like this:

```sql
SELECT
    t.TitleName
FROM title t
WHERE TitleID IN (
    SELECT TitleID
    FROM orderitem
    WHERE OrderID = 1034
    );
```

So now that you can use either of two logical operators with a subquery to get the same results, which should you choose? Well, the answer depends on whether you need to use a comparison operator too. If you have to use `>`, `>=`, `<`, or `<=`, you have to use `ANY` because `IN` doesn't allow those kinds of comparisons. If you don't need to use one of those comparison operators, however, you can use `IN`, which is used for these kinds of subqueries far more often than `=` `ANY`. Next, let's look at the opposite way to filter: excluding the results of our subquery.

### 11.2.2 The ALL and NOT IN operators

Suppose what we need is to find the names of titles that are *not* in order 1034, as shown in figure 11.3. We might say, "I would like the title name from the title table, and I would like the titles to not be in titles from order 1034." Just as we added the word *not* to our verbal declaration, we can do the same thing with our SQL:

```sql
SELECT
    t.TitleName
FROM title t
WHERE TitleID NOT IN (
    SELECT TitleID
    FROM orderitem
    WHERE OrderID = 1034);
```

##### Figure 11.3 The four titles not included in order 1034

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH11_F03_Iannucci.png)

Our sqlnovel database contains eight titles, and order 1034 includes four of them, as shown in figure 11.2. Now we know the other four titles that were not in that order. Our SQL statement is evaluating all the titles in order 1034 and then finding the titles in the title table that aren't any of those included in the order.

This brings us to another way we may be tempted to verbally declare the desired results of this query: "I would like the title name from the title table, and I would like the titles to not match any of the titles from order 1034." At first glance, we might think we can use the `ANY` operator to get these results by using it with the not-equal operator, `<>`:

```sql
SELECT
    t.TitleName
FROM title t
WHERE TitleID <> ANY (
    SELECT TitleID
    FROM orderitem
    WHERE OrderID = 1034
    );
```

Although this query will execute without error, it won't provide the desired results. This query is evaluating all the titles in the title table to see which ones don't match any of the titles in our subquery. Because our subquery contains more than one title, every title in the title table will be a match because at least one title in the subquery isn't the same.

##### Try it now

Execute this query, and notice that it returns every title in the title table.

To get the results we want, we need to use the `ALL` operator instead of `ANY` because we want titles that don't match *all* the titles in the subquery:

```sql
SELECT
    t.TitleName
FROM title t
WHERE TitleID <> ALL (
    SELECT TitleID
    FROM orderitem
    WHERE OrderID = 1034
    );
```

Executing this query produces the results shown in figure 11.3, which is what we intended. As with `IN` and `ANY`, the decision to use `NOT IN` or `ALL` comes down to whether a comparative operator is required.

##### NOTE

I haven't mentioned this topic yet, but be aware that by using subqueries, you're asking the RDBMS to execute two queries at the same time and evaluate the results of one against the other. Generally speaking, using subqueries requires more processing and memory, so when writing SQL, you should consider carefully whether using a subquery is necessary.

### 11.2.3 The EXISTS and NOT EXISTS operators

Two other operators can make subqueries more efficient than the ones we've seen so far: `EXISTS` and `NOT` `EXISTS`. These operators are appealing because they don't evaluate the values of every row in the subquery; rather, they check only for *any* matching rows. When a match for a value is found, other matches are not evaluated for equality or inequality.

First, we'll look at `EXISTS`, which is used similarly to the way we used `=` `ANY` and `IN` earlier to find the titles included in order 1034. The difference is that when we use `EXISTS`, we must include a kind of join in the `WHERE` clause of the subquery. Here's what this query would look like:

```sql
SELECT
    t.TitleName
FROM title t
WHERE EXISTS (
    SELECT TitleID
    FROM orderitem oi
    WHERE OrderID = 1034
        AND t.TitleID = oi.TitleID
    );
```

Executing this query provides the results shown in figure 11.2, returning the names of all the titles in order 1034. Notice that we used `EXISTS` in the `WHERE` clause, and now we have an additional line in the `WHERE` clause of the subquery: `AND` `t.TitleID` `=` `oi.TitleID`. This is where the evaluation for matching values in the subquery takes place. Because the evaluation occurs there, what we put in the `SELECT` clause of our subquery doesn't matter.

For this reason, you'll often see subqueries used with `EXISTS` in other people's SQL that has something that seems nonsensical in the `SELECT` clause, like this subquery, which has `SELECT` `1` in the `SELECT` clause:

```sql
SELECT
    t.TitleName
FROM title t
WHERE EXISTS (
    SELECT 1
    FROM orderitem oi
    WHERE OrderID = 1034
        AND t.TitleID = oi.TitleID
    );
```

##### Try it now

Execute the preceding query to see for yourself that it returns the results shown in figure 11.2. Try replacing the 1 in the `SELECT` clause of the subquery with any other value to see that the value there becomes irrelevant.

Also, we can use `NOT` `EXISTS` to find the titles that are *not* included in order 1034. Executing the following query returns the results shown in figure 11.3:

```sql
SELECT
    t.TitleName
FROM title t
WHERE NOT EXISTS (
    SELECT 1
    FROM orderitem oi
    WHERE OrderID = 1034
        AND t.TitleID = oi.TitleID
    );
```

Again, the main reason to use `EXISTS` or `NOT` `EXISTS` with subqueries is to query larger data sets because these operators provide better performance than `IN`/`NOT` `IN`, `ANY`, and `ALL`.

## 11.3 Subqueries in other parts of a query

So far in this chapter, we've looked only at subqueries used for filtering in the `WHERE` clause. But we can use subqueries in other clauses as well.

### 11.3.1 Subqueries in the FROM clause

We can write a query to return the results shown in figure 11.2 with a join in the `FROM` clause, for example. To do this, we move our subquery into a join in the `FROM` clause—in this case, using an inner join. We don't need any operators because we aren't evaluating the subquery for filtering. We're simply joining the results of the subquery, and any evaluation occurs with the `ON` part of the join.

Also, because our subquery doesn't have a name for the resulting data set, we'll need to use an alias so that we can join it to another table:

```sql
SELECT
    t.TitleName
FROM title t
INNER JOIN (
    SELECT TitleID
    FROM orderitem
    WHERE OrderID = 1034
    ) oisq
    ON t.TitleID = oisq.TitleID;
```

By moving the subquery to the `FROM` clause, we're treating our subquery results as though they were a table, with those results being joined to the title table by TitleID. The results of the subquery aren't a table, but they have to be computed by the RDBMS before we can determine if any rows from the results of our subquery can be joined to the title table.

What's interesting is that because the subquery is in the `FROM` clause, it can be used in our query like a table. As a result, we're no longer limited to having one column in our subquery, so we can add more columns to the subquery if necessary for joining or filtering purposes.

##### Try it now

Execute the preceding query, change `SELECT` `TitleID` to `SELECT` `TitleId`, `OrderID`, and execute that query as well.

We can also use a join in the `FROM` clause to find values that aren't in the subquery by using the `LEFT` `OUTER` `JOIN` method from chapter 9. We can use this kind of join with a filter on the null values in the second data set, which in this case is the subquery, to find values existing in the first table but not in the joined table:

```sql
SELECT
    t.TitleName
FROM title t
LEFT JOIN (
    SELECT TitleID
    FROM orderitem
 WHERE OrderID = 1034
    ) oisq
    ON t.TitleID = oisq.TitleID
WHERE oisq.TitleID IS NULL;
```

The results of this query are the same as those shown in figure 11.3.

### 11.3.2 Subqueries in the SELECT clause

A final way to use subqueries is the `SELECT` clause. We can get the same TitleNames shown in figure 11.2 by using a subquery in the `SELECT` clause, although we have to rearrange the query by switching the subquery from the filtering query on orderitem to the selection of the TitleName from the title table:

```sql
SELECT
    (
    SELECT TitleName
    FROM title t
    WHERE t.TitleID = oi.TitleID
    ) AS TitleName
FROM orderitem oi
WHERE oi.OrderID = 1034;
```

##### Warning

This approach is a highly unusual way to find this result set. I'm presenting this example only to show you what a subquery in the `SELECT` clause looks like. Writing subqueries in the `SELECT` clause is rarely the best way to write SQL.

We've seen several ways to use subqueries and the options they afford us in writing SQL. Note, however, that subqueries should be used thoughtfully because they often have a negative effect on query performance. The fact that each subquery executes an additional `SELECT` statement means our SQL statements with subqueries typically create more work for the RDBMS.

Chapter 12 looks at ways to group data sets to find calculations like the minimum and maximum values of those sets. First, though, you get to put your new subquery skills to use.

## 11.4 Lab

1.  Write a query using a subquery with `IN` to get the names of the title(s) in the only order placed by Joe Pagenaud.

2.  Look again at the queries in section 11.1, where we tried to find the orders placed after Margaret Montoya's order. Write a similar query to find the order ID, customer ID, and order date for any orders placed after all of Cora Daly's orders.

3.  Selecting `1` divided by `0` returns a null value. Could you use `SELECT` `1/0` instead of `SELECT` `TitleID` in the `SELECT` clause of the subquery of the first query in section 11.2.3 and still get the correct results?

## 11.5 Lab answers

1.  You have many ways to do this, depending on which queries you join in the subquery. Here's one way:

```sql
SELECT
    t.TitleName
FROM title t
INNER JOIN orderitem oi
    ON t.TitleID = oi.TitleID
WHERE oi.OrderID IN (
    SELECT
    oh.OrderID
    FROM orderheader oh
    INNER JOIN customer c
        ON oh.CustomerID = c.CustomerID
    WHERE c.FirstName = 'Joe'
        AND c.LastName = 'Pagenaud'
    );
```

2.  Your query may vary, but here's a way to find the intended order information:

```sql
SELECT
    OrderID,
    CustomerID,
    OrderDate
FROM orderheader
WHERE OrderDate > ALL (
    SELECT
        oh.OrderDate
    FROM orderheader oh
    INNER JOIN customer c
        ON oh.CustomerID = c.CustomerID
WHERE c.FirstName = 'Cora'
    AND c.LastName = 'Daly'
    );
```

3.  Yes, because the value or column used in the `SELECT` clause of a subquery is not evaluated by `EXISTS` or `NOT` `EXISTS`.

