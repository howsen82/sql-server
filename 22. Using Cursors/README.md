# 22 Using Cursors

In chapter 21, we explored making decisions in queries and learned how to make conditional evaluations. Using `IF` and `THEN` keywords allowed us to evaluate one or more values and then decide whether to do something else, such as insert a row of values into a table.

In this chapter, we'll look at other ways to evaluate data and make decisions in SQL, focusing primarily on cursors. Cursors enable us to evaluate a set of data one row or value at a time. Also, as we'll see, they have a bit of complexity, and there are important considerations regarding their use.

The use of cursors in MySQL is restricted to database objects containing prepared SQL, such as stored procedures. Because of this restriction, we'll look at some previously undiscussed features of variables and parameters before we dive into creating and using cursors.

## 22.1 Reviewing variables and parameters

We've used variables since chapter 13 and parameters since chapter 20. Although variables and parameters are similar in that they're placeholders for values, they have different properties relative to their use. The following sections show how we can use some of these properties.

### 22.1.1 Variables inside stored procedures

Chapter 13 briefly mentioned the way variables are declared in MySQL and how they differ in other relational database management systems (RDBMSes). In case you don't remember, here's the warning from that chapter:

##### Warning

This method of declaring variables in MySQL isn't universal. When you use a different RDBMS, such as SQL Server or PostgreSQL, first you have to declare a user-defined variable using the `DECLARE` keyword; then you assign it a specified data type.

Interestingly, in MySQL we have to use the more common method (with the `DECLARE` keyword noted in the warning) of operating inside a stored procedure. Let's look at an example. If we want to declare a variable to hold a value for TitleID from the title table outside a stored procedure, we'd do so like this:

```sql
SET @TitleID = 101;
```

Inside a stored procedure, however, we'd have to declare the variable and its data type using the `DECLARE` keyword, like this:

```sql
DECLARE _TitleID int;
```

Now we have a few options for assigning a value (such as 101) to our variable inside the stored procedure. The first option is using the `SET` keyword like this:

```sql
SET _TitleID = 101;
```

The `SET` keyword gives us some options for setting this value dynamically via a subquery. Here's an example:

```sql
SET _TitleID =
    (SELECT TitleID
    FROM title
    WHERE TitleName = 'Pride and Predicates');
```

We have another option for assigning a specific value to our variable. We can use a default value using the `DEFAULT` keyword:

```sql
DECLARE _TitleID int DEFAULT 101;
```

This final option is what we'll use in cursor examples for the rest of this chapter.

### 22.1.2 Output parameters

Chapter 20 noted that parameters in stored procedures can be used for either input or output. So far, we've used only input parameters, which allow us to pass a value into a stored procedure, by declaring them with the `IN` keyword:

```sql
CREATE PROCEDURE GetSomeData(
    IN _TitleName varchar(50)
    )
```

Declaring a parameter for output lets us take a value determined inside the stored procedure and its SQL and pass it to a script or even another stored procedure. We do this fairly intuitively with the `OUT` keyword, as in this example:

```sql
CREATE PROCEDURE GetSomeData(
    OUT _TitleName varchar(50)
    )
```

The cursor examples in this chapter use output parameters, so you'll have several chances to get comfortable with them and their use.

## 22.2 Cursors

At its most basic level, a *cursor* is a database object that steps through the results of a `SELECT` query, allowing you to retrieve and, if you desire, manipulate data one row at a time. Much as a cursor in an electronic document tells you where you're working, a cursor is a row pointer that enables you to loop through the result set of a query, processing individual rows for whatever intended reason.

Although cursors are simple to explain, they can be intimidating to use due to the complexity of their parts relative to other objects we've used, such as views and stored procedures. By that, I mean we can create simple views and stored procedures but not simple cursors. Even the most basic cursors may look a bit intimidating at first. To make cursors more understandable, let's examine their four core components.

### 22.2.1 Anatomy of a cursor

No matter how simple or complex they are, every cursor has four parts that include these descriptive keywords:

* `DECLARE`—Just as we used `DECLARE` earlier in this chapter to create a variable inside a stored procedure, we'll use it to create our cursor. The `DECLARE` part will contain the `SELECT` query that defines the set of data our cursor will use.
* `OPEN`—After we create the cursor, we must open it. Although the `DECLARE` part defined the data set for the cursor to use, that `SELECT` query won't execute until we open the query in this part. Here, the cursor gets the results of our `SELECT` query and holds them in server memory while we use the cursor.
* `FETCH`—The `FETCH` keyword retrieves rows from the data set one row at a time. This part is the main part of the cursor, where we populate variables, modify data, and do whatever else we intend to do with it. We work with that single row until the next time we `FETCH` another row as we loop through the result set, and we stop fetching rows when we reach the end of the data set.
* `CLOSE`—When we determine that we're done fetching and evaluating rows in our data set, we `CLOSE` the cursor, which releases the contents of the cursor from server memory.

##### NOTE

These four parts must exist for a cursor to work, and they must be in this order.

##### Warning

Although MySQL doesn't, other RDBMSes may require you to deallocate a cursor when you finish using it. Please refer to the documentation of your specific RDBMS to see whether this step is required for any cursors you write outside MySQL.

### 22.2.2 Creating a cursor

Suppose that we want to determine the quantity of titles sold at the price listed in the title table, with no promotional discounts applied. We could write a stored procedure that uses a cursor to step through each order, checking it for any titles sold at the price listed in the title table. As our cursor goes through each order, it can keep a running total of the quantity of titles sold at the list price in an output parameter, which is returned to us with the final quantity of titles sold at the list price.

I'll go through the parts of this cursor later in this section. For now, when you take your first look at this stored procedure, try to see whether you can identify the four main parts of the cursor:

```sql
DELIMITER //

CREATE PROCEDURE GetTitleTotalQuantitySoldListPrice(
    OUT _TotalQuantitySold int
    )
BEGIN

DECLARE _Done boolean DEFAULT FALSE;

DECLARE _OrderID int;

DECLARE AllOrders CURSOR FOR

SELECT OrderID
FROM orderheader;

DECLARE CONTINUE HANDLER FOR NOT FOUND SET _Done = TRUE;

SET _TotalQuantitySold = 0;

OPEN AllOrders;

GetOrders: LOOP

    FETCH AllOrders INTO _OrderID;

    SET _TotalQuantitySold = _TotalQuantitySold +

        (SELECT COALESCE(SUM(Quantity),0)
        FROM title t
        INNER JOIN orderitem oi
             ON t.TitleID = oi.TitleID
             AND t.Price = oi.ItemPrice
        WHERE oi.OrderID = _OrderID
        );

        IF _Done = TRUE THEN

            LEAVE GetOrders;
        END IF;

    END LOOP GetOrders;

    CLOSE AllOrders;

END //

DELIMITER ;
```

That's a lot of SQL to evaluate, so we'll examine it one bit at a time. We start by changing the delimiter to two forward slashes so we can use semicolons inside our stored procedure:

```sql
DELIMITER //
```

Next, we will use `CREATE` `PROCEDURE` to say we want to make a procedure named GetTitleTotalQuantitySoldListPrice. Note that our procedure has a parameter called `_TotalQuantitySold`, which not only has a data type of `int` but also is used as an output parameter. We know this because the word `OUT` precedes the parameter name:

```sql
CREATE PROCEDURE GetTitleTotalQuantitySoldListPrice(
    OUT _TotalQuantitySold int
    )
BEGIN
```

Then we declare two variables: one for `_Done` and one for `_OrderID`. The `_OrderID` variable will be used for the OrderIDs that we'll evaluate one by one in the cursor. The `_Done` variable will determine whether we've finished evaluating all rows. This variable uses a new data type called `boolean`, which means that its value is either `TRUE` or `FALSE`. We assign a default value of `FALSE` at the start of the stored procedure because we haven't finished (or even started) evaluating a data set in our cursor:

```sql
DECLARE _Done boolean DEFAULT FALSE;
DECLARE _OrderID int;
```

Now we have the first part of our cursor: the `DECLARE` part. We declared our cursor with the name `AllOrders`, and our data set will include every OrderID in the orderheader table:

```sql
DECLARE AllOrders CURSOR FOR

SELECT OrderID
FROM orderheader;
```

Then we use another `DECLARE` statement to tell the RDBMS that it should handle the condition of no more rows to evaluate (`NOT FOUND SET`) by setting our `_Done` variable, which indicates that we're done using the cursor, to `TRUE`. This allows us to break out of the loop and stop fetching rows:

```sql
DECLARE CONTINUE HANDLER FOR NOT FOUND SET _Done = TRUE;
```

Because MySQL doesn't allow us to set a default value for parameters, we're setting the value of `_TotalQuantitySold` to 0 as a starting value. Later, we'll increase this value incrementally as we find titles that were sold at the list price:

```sql
SET _TotalQuantitySold = 0;
```

Now we get to the second part of the cursor, where we `OPEN` the cursor. The query used in the `DECLARE` part executes, and the resulting data set is stored in memory for use by the cursor:

```sql
OPEN AllOrders;
```

Next, we use a `LOOP` statement named `GetOrders` to loop through the data set. This `LOOP` statement is a requirement for cursors in MySQL:

```sql
GetOrders: LOOP
```

##### Note

Not all RDBMSes require you to use `LOOP` with a cursor, so this part may be unnecessary in another RDBMS. Consult the documentation of the RDBMS you're using to understand the various requirements for any cursor.

With the cursor open, we retrieve the first row of values with the `FETCH` part of our query. In the case of our cursor, we're selecting only the OrderID column of values, so we're fetching the first value for OrderID and assigning it to the `_OrderID` variable:

```sql
    FETCH AllOrders INTO _OrderID;
```

Now that we have a value for `_OrderID`, we can evaluate the order to see whether it includes any titles that were sold for the price listed in the title table. If it does, we'll increase the value of `_TotalQuantitySold` by the quantity of titles that were sold for the list price in the order. If it doesn't, using `COALESCE` will allow us to increment the value for `_TotalQuantitySold` by zero. Remember that the query used by the cursor is evaluated in a loop, so we'll repeat it for every OrderID in the set from the `DECLARE` part of our cursor:

```sql
    SET _TotalQuantitySold = _TotalQuantitySold +

        (SELECT COALESCE(SUM(Quantity),0)
        FROM title t
        INNER JOIN orderitem oi
             ON t.TitleID = oi.TitleID
             AND t.Price = oi.ItemPrice
        WHERE oi.OrderID = _OrderID
        );
```

Remember when we declared our handler to set the value of `_Done` to `TRUE` if it reached the end of the results in the cursor? Well, if that's the case here, we'll use an `IF` statement to exit the loop with the `LEAVE` keyword:

```sql
        IF _Done = TRUE THEN

            LEAVE GetOrders;
        END IF;
```

##### Note

`LEAVE` is another keyword used in MySQL, but it's not used to exit cursor loops in other RDBMSes. Again, consult the documentation for any RDBMS you're using to determine how to exit a cursor loop.

Our loop process can't go on forever, so here, we define the end of the loop. If we haven't exited the loop via the preceding statement, we fetch another value for OrderID at the beginning of the loop:

```sql
    END LOOP GetOrders;
```

If we've reached this point, we've exited the loop, so we're done using our cursor. If we're done using our cursor, we need to close it and release the contents from memory. To do this, we use the `CLOSE` keyword, which is the fourth and final part of our cursor:

```sql
    CLOSE AllOrders;
```

All the work with the cursor is done now, so we need to note the end of the stored procedure with the `END` keyword and the nonstandard statement delimiter noted at the beginning of our script:

```sql
END //
```

Finally, we change the statement delimiter back to the standard semicolon:

```sql
DELIMITER ;
```

If we execute all that SQL, we can call the stored procedure with the output parameter, which we can capture in a variable named `@TotalQuantitySold`. Then we can select the value of the variable to see the total quantity of titles sold at list price, with the value shown in figure 22.1:

```sql
CALL GetTitleTotalQuantitySoldListPrice(@TotalQuantitySold);
SELECT @TotalQuantitySold AS TotalQuantitySold;
```

##### Figure 22.1 The TotalQuantity of titles sold at list price, determined by the stored procedure we wrote, which uses a cursor to determine this value

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH22_F01_Iannucci.png)

##### Try it now

Create the stored procedure GetTitleTotalQuantitySoldListPrice and execute the preceding query to verify the total quantity of titles sold at list price.

Even the most basic cursors can appear to be complicated, but I hope that this walkthrough of a cursor used inside a stored procedure cleared up any confusion. If you're still a bit confused, you may be encouraged to know that less-complicated alternatives can give you much of the same functionality you'd get from a cursor.

## 22.3 Alternatives to cursors

A common replacement for a cursor in SQL is a `WHILE` loop, which requires a lot less language to do the same row-by-row evaluation while looping through a set of data.

### 22.3.1 Using WHILE

What makes the `WHILE` loop simpler is the fact that we don't have to open or close our data set. We don't even need a data set for a `WHILE` loop—only a condition for the `WHILE` statement that must be met to determine whether to continue the loop. Here's how we'd rewrite our GetTitleTotalQuantitySoldListPrice stored procedure to use a `WHILE` loop instead of a cursor:

```sql
DROP PROCEDURE GetTitleTotalQuantitySoldListPrice;

DELIMITER //

CREATE PROCEDURE GetTitleTotalQuantitySoldListPrice(
    OUT _TotalQuantitySold int
    )
BEGIN

DECLARE _OrderID int;

SET _TotalQuantitySold = 0;
SET _OrderID = (SELECT MIN(OrderID) FROM orderheader);

WHILE _OrderID IS NOT NULL DO
    SET _TotalQuantitySold = _TotalQuantitySold +

        (SELECT COALESCE(SUM(Quantity),0)
        FROM title t
        INNER JOIN orderitem oi
             ON t.TitleID = oi.TitleID
             AND t.Price = oi.ItemPrice
        WHERE oi.OrderID = _OrderID
        );

    SET _OrderID =
        (SELECT MIN(OrderID)
        FROM orderheader
        WHERE OrderID > _OrderID);

    END WHILE;

END //

DELIMITER ;
```

Let's examine the new parts so that we understand what we're doing. First, instead of fetching the first value into our `_OrderID`, variable, we used a `SET` statement, which uses the `MIN` function to select the minimum value from the orderheader table. That value is effectively the same as the first value chosen by the preceding cursor:

```sql
SET _OrderID = (SELECT MIN(OrderID) FROM orderheader);
```

Then we declare our `WHILE` statement, which says to keep looping through the SQL contained in the loop until `_OrderID` is `NULL`. We start the loop with a new keyword, `DO`:

```sql
WHILE _OrderID IS NOT NULL DO
```

##### Note

Although the `DO` keyword is used in MySQL, other RDBMSes often start with `BEGIN`. I know it may be frustrating to keep being warned about the differences in SQL use among RDBMSes, but consult the appropriate documentation to avoid syntax errors.

This next part should look familiar. It's the same logic we used in our cursor to incrementally add to the `_TotalQuantitySold` parameter that we used in our cursor:

```sql
    SET _TotalQuantitySold = _TotalQuantitySold +

        (SELECT COALESCE(SUM(Quantity),0)
        FROM title t
        INNER JOIN orderitem oi
             ON t.TitleID = oi.TitleID
             AND t.Price = oi.ItemPrice
        WHERE oi.OrderID = _OrderID
        );
```

Next, we'll increment the value of `_OrderID` to the next-highest value, using logic similar to what we used to grab the first minimum value. The difference is that now we're grabbing the minimum value that's higher than the current value, which is the next value for OrderID in the orderheader table:

```sql
    SET _OrderID =
         (SELECT MIN(OrderID)
         FROM orderheader
         WHERE OrderID > _OrderID);
```

Finally, we end the SQL included in the `WHILE` loop with `END WHILE`:

```sql
    END WHILE;
```

This code is less SQL than we used for our cursor, but it effectively does the same thing. Because the cursor and the `WHILE` loop do the same thing, however, they could create the same problem: blocking. *Blocking* happens when a query locks resources such as rows in a table, causing other queries that require the same resources to wait until the first query completes its execution.

Although all the SQL we've written and executed so far has been for our MySQL database, of which we're the only users, the SQL you write outside this book will be for a database with tens, hundreds, or even thousands of users. Depending on database settings that you may not control, your cursor or `WHILE` loop in a database with more connections could cause blocking for other users, making their queries take longer or even fail if the connection has to wait too long. One way to work around this problem is to use temporary tables.

##### Tip

Many RDBMSes have options for cursors beyond what we've used to reduce the chance of blocking. But the default options for cursors often result in blocking.

### 22.3.2 Temporary tables

*Temporary tables* are useful because they allow us to copy a data set that might be heavily used to a separate table that exists only as long as our connection to the database exists. When the connection is closed, the temporary tables are dropped from the database. More pertinent to cursors and `WHILE` loops, we can use them with no chance of blocking because they can be used only by the queries in our connection.

The syntax for creating a temporary table in MySQL is almost identical to the syntax we used to create tables in chapter 18. The only difference is that we add the word `TEMPORARY` between `CREATE` and `TABLE`.

##### Note

Although temporary tables exist for nearly every RDBMS, the syntax for creating them isn't universal. I hope you aren't tired of seeing this message, but consult the relevant documentation.

To prevent blocking, we could create a temporary table inside our stored procedure, populate the table with the range of values we plan to use, and then direct our `WHILE` loop (or cursor) to loop through the temporary table.

For the preceding version of GetTitleTotalQuantitySoldListPrice, here's how we could drop the existing stored procedure and then re-create it using a temporary table named orderheadertemp that replaces our use of the orderheader table:

```sql
DROP PROCEDURE GetTitleTotalQuantitySoldListPrice;

DELIMITER //

CREATE PROCEDURE GetTitleTotalQuantitySoldListPrice(
    OUT _TotalQuantitySold int
    )
BEGIN
DECLARE _OrderID int;

SET _TotalQuantitySold = 0;

CREATE TEMPORARY TABLE orderheadertemp (OrderID int);

INSERT orderheadertemp (OrderID)
SELECT OrderID
FROM orderheader;

SET _OrderID = (SELECT MIN(OrderID) FROM orderheadertemp);

WHILE _OrderID IS NOT NULL DO
    SET _TotalQuantitySold = _TotalQuantitySold +

        (SELECT COALESCE(SUM(Quantity),0)
        FROM title t
        INNER JOIN orderitem oi
             ON t.TitleID = oi.TitleID
             AND t.Price = oi.ItemPrice
        WHERE oi.OrderID = _OrderID
        );

    SET _OrderID =
        (SELECT MIN(OrderID)
        FROM orderheadertemp
        WHERE OrderID > _OrderID);

    END WHILE;

END //

DELIMITER ;
```

Temporary tables are wonderful tools to use for more than preventing blocking. We can use them to hold data sets that are used repeatedly in a SQL script, and we can use them to simplify complex queries by separating them into smaller, more efficient queries. But after all this talk of cursors, `WHILE` loops, and temporary tables, we should take a moment to ask a question: Is all this even necessary?

## 22.4 Considerations for using cursors

If you've read all the preceding chapters and completed the exercises, there's a good chance that you've thought of a better way to find the total quantity of titles sold at list price. In that case, you're correct. You could have handled this request much more simply without using a cursor or a `WHILE` loop:

```sql
SELECT COALESCE(SUM(Quantity),0) AS TotalQuantitySold
FROM title t
INNER JOIN orderitem oi
    ON t.TitleID = oi.TitleID
    AND t.Price = oi.ItemPrice;
```

Unfortunately, cursors and `WHILE` loops have a common problem: they are usually inferior to other options in SQL. Cursors and `WHILE` loops are inferior solutions for most query requests because the nature of evaluating data sets row by row is the opposite of the way that an RDBMS is designed to evaluate data, which is to use data sets.

### 22.4.1 Thinking in sets

Starting with our first query, everything we've seen in this book up to the last part of chapter 21 involves set-based programming. *Set-based programming* tells the RDBMS what data set or data sets we want to evaluate; from there, we let the RDBMS figure out the best way to complete the query.

We used set-based programming with our SQL queries over and over until we started working with the `IF`, `THEN`, and `ELSE` keywords, along with cursors and `WHILE` loops. That type of programming is called *procedural programming*, which gives the RDBMS specific instructions about what to do and how to do it. Procedural programming is quite common for programming languages other than SQL.

One reason why cursors are so long and wordy is that we need to tell the RDBMS every step to take to get a cursor to evaluate data. Unfortunately, because we're telling the RDBMS what to do, taking a procedural approach in SQL often results in slow performance, extensive blocking, and the consumption of more server resources than a query with a set-based approach would use.

### 22.4.2 Thinking about cursor use

I don't mean to say that you should *never* use cursors, although it's possible to solve nearly every query request without using them. But as you near the final chapters of this book, I want to encourage you to use your total knowledge of SQL and look at cursors with a bit of skepticism.

Knowing how to construct a cursor can be useful when you have a request with no other solution than to evaluate each row in a set individually. But those cases are rare, so even when you think you need to use a cursor, take a moment to ask whether a set-based solution exists.

In terms of evaluating existing code, look at any cursor you encounter as a potential opportunity to improve performance by replacing it with set-based programming. You'll likely encounter cursors frequently in existing stored procedures, as many folks with experience doing procedural programming in other languages often lean on cursors in SQL instead of the set-based methods you've learned throughout this book. Use your knowledge not only to reduce the complexity involved in cursors but also to improve the performance of stored procedures and reduce the resources they require from the server.

I hope you're excited about the prospect of reviewing someone else's SQL, because that's exactly what you'll do in chapter 23.

## 22.5 Lab

This lab is a bit different from earlier labs. Here, you'll consider a few scenarios and try to determine whether you need to use a cursor to retrieve the data:

1.  Evaluate every TitleName in the title table to determine the quantity of each title ordered by customers in California. This query should include customers with a value of CA for the State column in the customer table.

2.  Evaluate every CustomerID in the customer table to determine whether the customer purchased the title Pride and Predicates. Return `Yes` if they did and `No` if they didn't in a column named OrderedPrideAndPredicates.

3.  Evaluate each order to determine whether it was the first that a customer placed. Return the OrderID, CustomerID, and OrderDate of all orders that were the first by any customer.

4.  Evaluate every CustomerID in the customer table to see whether the customer placed an order in the past year. If a customer placed such an order, execute a stored procedure named CreateThankYouMessage. This stored procedure, which contains a single CustomerID parameter, creates a message to be sent to the customer.

## 22.6 Lab answers

1.  You don't need a cursor to determine this information. You can find the requested data set with a query like this one, which uses a subquery to collect the quantity of titles ordered by customers in California:

```sql
SELECT
    t.TitleName,
    COALESCE(SUM(x.Quantity),0) AS QuantityFromCA
FROM title t
LEFT JOIN (
    SELECT
        oi.TitleID,
        oi.Quantity
    FROM orderitem oi
    INNER JOIN orderheader oh
        ON oi.OrderID = oh.OrderID
    INNER JOIN customer c
        ON oh.CustomerID = c.CustomerID
    WHERE c.State = 'CA'
    ) x
    ON t.TitleID = x.TitleID
GROUP BY t.TitleName;
```

2.  You don't need a cursor for this task either. You can use a subquery and a `CASE` statement to determine the presence of data in the subquery. Be careful to use `COALESCE` or some other way to evaluate something other than NULL, because NULL can't be evaluated in a `CASE` statement. The following example defaults NULL values to 0 because no CustomerID of 0 exists:

```sql
SELECT
    c.CustomerID,
    CASE COALESCE(x.CustomerID,0)
        WHEN 0 THEN 'No'
        ELSE 'Yes'
        END AS OrderedPrideAndPredicates
FROM customer c
LEFT JOIN (
    SELECT oh.CustomerID
    FROM orderheader oh
    INNER JOIN orderitem oi
        ON oh.OrderID = oi.OrderID
    INNER JOIN title t
        ON oi.TitleID = t.TitleID
    WHERE t.TitleName = 'Pride and Predicates'
    GROUP BY oh.CustomerID
    )x
    ON c.CustomerID = x.CustomerID
ORDER BY c.CustomerID;
```

3.  Again, you don't need a cursor. You can use another subquery to determine the first OrderID for each CustomerID and then select the desired rows from the subquery with an `INNER JOIN`:

```sql
SELECT
    oh.OrderID,
    oh.CustomerID,
    oh.OrderDate
FROM orderheader oh
INNER JOIN (
    SELECT
        ohf.CustomerID,
        MIN(ohf.OrderID) AS FirstOrderID
    FROM orderheader ohf
    GROUP BY ohf.CustomerID
    ) x
    ON oh.OrderID = x.FirstOrderID;
```

Also, in each of these three exercises, you could use the SQL in the subqueries to populate a temporary table and then join that temporary table instead of joining the subquery. The point is that you have choices beyond using a cursor to achieve the desired results.

4.  This case is one of the few times you'd need to use a cursor. You have a data set of CustomerID values, but you can't use set-based programming to complete the request because you can provide only one value at a time to the CreateThankYouMessage stored procedure.

