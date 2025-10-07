# 21 Making Decisions in Queries

Now that you know how to add, update, or remove data from a table, let's look at some of the tools SQL provides for making decisions in queries and stored procedures. What if you want to group data and return a value of 0 if a value of the `SUM` is NULL, for example? What if you want to make the output of a query dependent on some condition or to evaluate parameters in a stored procedure and provide conditional feedback in the output? This chapter looks at all these scenarios and more.

## 21.1 Conditional functions and expressions

Do you recall that you've already used one function in a conditional expression a few times? That function is `COALESCE`. You used it in chapter 15 to concatenate the full names of authors and in chapter 20 to handle NULL values for TitleName. In the first example, `COALESCE` allowed you to avoid a result of NULL for concatenated full names in cases when there were null values for the MiddleName of any authors.

### 21.1.1 COALESCE function

In chapter 15, we used the following query, which provided two values to `COALESCE` to evaluate:

```sql
SELECT CONCAT(FirstName, ' ', COALESCE(MiddleName, ''), ' ', LastName)
    AS AuthorName
FROM author;
```

The `COALESCE` function evaluates any number of expressions from left to right, returning the first non-null values it finds. Because the MiddleName column of the author table was the first value provided, the `COALESCE` function evaluated the MiddleName value for each author and then determined whether the MiddleName value was NULL. For each row in which the value was not NULL, the MiddleName value was used in the context of the query. For each row in which the value was NULL, the second value—an empty string represented by two single quotes ('')—was used for concatenation.

The `COALESCE` function has more functionality than we used in the preceding query because we can give it more than two expressions to evaluate for NULL values. Here's a simple example that uses three expressions, of which the first two are NULL:

```sql
SELECT COALESCE(NULL, NULL, 'I am not null!') AS CoalesceTest;
```

The `COALESCE` function evaluated the first two expressions, determined them to be NULL values, and returned the third expression—the string 'I am not null!'—as the first non-NULL expression. We could have had the `COALESCE` function evaluate more than three expressions, but as soon as one expression is determined to be not NULL, all subsequent expressions are ignored.

##### Try it now

Execute the following query using the `COALESCE` function:

```sql
SELECT COALESCE(NULL, NULL, 'I am not null!', 'I am ignored!')
    AS CoalesceTest;
```

### 21.1.2 IFNULL function

Another common function is used for evaluating nulls: `IFNULL`. The `IFNULL` function is nearly identical in use to the `COALESCE` function, the main exception being that it's limited to evaluating only two expressions. The first expression is evaluated for null values, and if the value is determined to be NULL, the second expression is returned. Here's an example:

```sql
SELECT IFNULL(NULL, 'I am not null!') AS IfNullTest;
```

##### Note

The `IFNULL` function doesn't exist in all relational database management systems (RDBMSes). In Microsoft Access and SQL Server, you'll use the `ISNULL` function instead, and in Oracle, you'll use the `NVL` function. Although these functions have different names, you use them the same way as `IFNULL`.

Although we've used literal values in these examples, `COALESCE` and `IFNULL` are often used with calculations that might include null values. Let's consider a common scenario in which we want to see a list of all title names and determine whether they're included in any orders.

First, we'll consider the tables we need to use in this query. We need the title table because it has TitleNames. We also need the orderitem table, which contains the Quantity column, representing the quantity of each item ordered in each row. Finally, we need the orderheader table because it relates to both the title and orderitem tables with the TitleID and OrderID columns, respectively.

The way we join these tables is crucial to our output. We want to start with the title table because we want the total quantity sold for each title, but we need to use `LEFT JOIN`s to join the other tables because some titles may not have been included in any orders. If we used `INNER` `JOIN`s to join all the tables, we'd get a result set that included only titles that were included in orders—not what we wanted from this query.

We also want to group by TitleName from the title table and use a `SUM` function to get the sum of the Quantity column from orderitem for each TitleName. Our query will look something like the following code snippet (results shown in figure 21.1):

```sql
SELECT
    t.TitleName,
    SUM(oi.Quantity) AS TotalQuantity
FROM title t
LEFT JOIN orderitem oi
    ON t.TitleID = oi.TitleID
LEFT JOIN orderheader oh
    ON oh.OrderID = oi.OrderID
GROUP BY t.TitleName
ORDER BY t.TitleName;
```

##### Figure 21.1 The TitleName and the total quantity of TitleName included in orders. TitleName values that weren't included in orders are represented by NULL.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH21_F01_Iannucci.png)

If you executed the queries in chapter 16 that added the extra titles, some rows in your results should have NULL for TotalQuantity. In a sales report, NULL typically isn't what readers expect, so make a minor adjustment to your query to add `IFNULL`, which returns a value of 0 for any title that isn't included in an order (results shown in figure 21.2):

```sql
SELECT
    t.TitleName,
    IFNULL(SUM(oi.Quantity),0) AS TotalQuantity
FROM title t
LEFT JOIN orderitem oi
    ON t.TitleID = oi.TitleID
LEFT JOIN orderheader oh
    ON oh.OrderID = oi.OrderID
GROUP BY t.TitleName
ORDER BY t.TitleName;
```

##### Figure 21.2 The TitleName and the total quantity of TitleName included in orders. TitleName values that weren't included in orders are represented by a value of 0 instead of NULL, due to the use of the `IFNULL` function.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH21_F02_Iannucci.png)

Now our results have a value of 0 instead of NULL for any title that wasn't included in any order, which is a more useful indicator of titles included in orders.

##### Tip

Because `COALESCE` has more functionality and is supported by every RDBMS, use this function instead of `IFNULL` or `ISNULL` to evaluate nulls. We'll be using `COALESCE` throughout this book instead of `IFNULL`.

### 21.1.3 CASE expression

`COALESCE` and `IFNULL` help us evaluate expressions for null values. But what if we need to evaluate for something other than null values or evaluate for different conditions? In these situations, we can use the `CASE` expression.

The `CASE` expression, often referred to in queries as a `CASE` statement, is a more powerful tool than `COALESCE` and `IFNULL` because it allows us to evaluate conditions and return different values based on those conditions. `CASE` lets us make choices on the values that are returned, using logic that emulates the English language. If we want to find titles with prices that are $7.95 and return a value that confirms whether they are $7.95, for example, we might say something like this: "I would like title name and price from the title table. When the price is $7.95, I want to say, 'This title is $7.95.' Otherwise, I want to say, 'This title is not $7.95.' "

We can use a `CASE` expression to accomplish the intention of the last two sentences. The structure of our `CASE` expression has a few rules:

* It must start with the keyword `CASE`.
* It must contain one or more conditions for equality that say `WHEN` (some value or expression) `THEN` (the desired value). Because this test is for equality, it won't evaluate NULL values.
* We can use `ELSE` to account for any value that doesn't meet the `WHEN` conditions, but `ELSE` can be used only after all `WHEN` conditions. Most `CASE` expressions include `ELSE` to account for unknown or NULL values.
* The `CASE` expression must conclude with the keyword `END`. If the `CASE` expression is used in the `SELECT` part of our query, we typically want to use an alias for the column name for readability.

This expression may sound a little complicated, but its use is intuitive. Here's what our SQL from the preceding example looks like when we use `CASE` (results shown in figure 21.3):

```sql
SELECT
    TitleName,
    Price,
    CASE Price
        WHEN 7.95 THEN 'This title is $7.95.'
        ELSE 'This title is not $7.95.'
        END AS IsPrice795
FROM title;
```

##### Figure 21.3 The TitleName and Price for all titles, as well as a column aliased as `IsPrice795`. The values in column `IsPrice795` are the results of an evaluation of the price by a `CASE` expression.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH21_F03_Iannucci.png)

As I mentioned, the evaluation in a `CASE` expression can be for another expression, which may not necessarily be a value like values in a column. An expression could be the result of anything from concatenating two or more columns to performing a mathematical calculation. We can evaluate either of those examples or any other expression with a `CASE` expression.

Let's look at an example using the `ROUND` function (first discussed in chapter 15). If we want an expression to represent the integer values of a number such as the price of a title, we use the expression `ROUND(Price,` `0)` to find the nearest integer value.

We can modify the preceding query to look for books with a price of about $8 by using the expression `ROUND(Price,` `0)` and then use a `CASE` expression to return a factual statement about the price (results shown in figure 21.4):

```sql
SELECT
    TitleName,
    Price,
    CASE ROUND(Price, 0)
        WHEN 8 THEN 'This title is around $8.'
        ELSE 'This title is not around $8.'
        END AS IsPriceAround8Dollars
FROM title;
```

##### Figure 21.4 The TitleName and Price for all titles, as well as a column aliased as `IsPriceAround8Dollars`. The values in column `IsPriceAround8Dollars` are the result of an evaluation of the expression `ROUND(Price,` `0``)` using a `CASE` expression.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH21_F04_Iannucci.png)

The preceding two queries are examples of using simple `CASE` expressions, which means that we're evaluating possible values to match a single expression. We can also use a searched `CASE` expression for more comprehensive evaluations, such as ranges of data values. With a searched `CASE` expression, we'll evaluate one or more other expressions to see whether they're true or false.

We can modify the preceding query to search for ranges of price values and see whether a price is less than, equal to, or more than $8 by using a searched `CASE` statement with the following query (results shown in figure 21.5):

```sql
SELECT
    TitleName,
    Price,
    CASE
        WHEN Price < 8.00 THEN 'This title is less than $8.00.'
        WHEN Price = 8.00 THEN 'This title is $8.00.'
        WHEN Price > 8.00 THEN 'This title is more than $8.00.'
        END AS IsPriceAround8Dollars
FROM title;
```

##### Figure 21.5 The TitleName and Price for all titles, as well as a column aliased as `IsPriceAround8Dollars`. The values in column `IsPriceAround8Dollars` are the result of a searched `CASE` expression evaluating whether the Price value is less than, equal to, or more than $8.00.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH21_F05_Iannucci.png)

The searched `CASE` expressions evaluated for being true or false are known as *Boolean expressions*. I realize that this chapter has talked about several kinds of expressions, complicated by `CASE` expressions evaluating other expressions. Please remember that those expressions evaluated in the `WHEN` parts of the preceding query are Boolean; we'll use them again later in this chapter.

##### Note

Although we've used `CASE` expressions only in the `SELECT` clause, if you need this kind of decision-making logic elsewhere in your queries, you can use `CASE` expressions in other clauses, including `WHERE`, `HAVING`, and `ORDER` `BY`.

## 21.2 Decision structures

Functions and expressions aren't the only tools we have in SQL for evaluation and decision-making. We can also use several keywords to decide whether we'll even execute SQL statements.

Nearly every RDBMS has keywords you can use to control decision-making. If you've ever used a programming language, the good news is that those keywords should be familiar. If you're new to programming, don't worry; the keywords are very intuitive.

### 21.2.1 IF and THEN

First, we'll look at the one keyword necessary to start any decision-making: `IF` . This keyword is the starting point for any *decision structure*, which is how we refer to any SQL we write that involves deciding whether we want to execute a statement. We'll base our decisions on the same Boolean conditions we used in section 21.1.3. This means that if a condition is true, we want the included SQL to execute, and if that condition is false, we don't want it to execute.

Decision structures are commonly used within stored procedures, so let's look at a simple example of using the `IF` keyword to make a decision inside a stored procedure that adds a row to the promotion table. We can write a stored procedure to add a row for a new PromotionCode, but we can create a decision structure to avoid writing the row if a value for PromotionCode isn't provided.

Before we look at the entire stored procedure, let's look at the individual parts of the SQL we'll use inside the stored procedure to determine whether a PromotionCode value exists. Here's the first part of the stored procedure:

```sql
CREATE PROCEDURE AddPromotion (
    IN _PromotionID int,
    IN _PromotionCode varchar(10),
    IN _PromotionStartDate datetime,
    IN _PromotionEndDate datetime
)
BEGIN
```

Looking at this part of the stored procedure, we see a name (`AddPromotion`) and four input parameters: `_PromotionID`, `_PromotionCode`, `_PromotionStartDate`, and `_PromotionEndDate`. The data types used for these parameters are the same as the corresponding columns in the promotion table. We also have the `BEGIN` keyword after we declare the parameters to indicate where the stored procedure begins doing what we want it to do.

##### Tip

Always create parameters with the same data types as any columns they'll read or write values to. Using a different data type will cause the RDBMS to do more work and negatively affect query performance. If you don't know the data types of the columns, you can always use `SHOW` `COLUMNS` (discussed in chapter 20) to determine column data types. Although `SHOW` `COLUMNS` exists only in MySQL, every RDBMS has similar keywords to help you view the data types of the columns of any table.

Next, let's look at the SQL for the rest of the stored procedure:

```sql
IF _PromotionCode IS NOT NULL THEN
    INSERT INTO promotion (
    PromotionID,
    PromotionCode,
    PromotionStartDate,
    PromotionEndDate
    )
    SELECT
        _PromotionID,
        _PromotionCode,
        _PromotionStartDate,
        _PromotionEndDate
        ;
END IF;
END
```

Here, we have the `IF` keyword, which says we want to determine whether the condition of `_PromotionCode` `IS` `NOT` `NULL` is true. If a value other than NULL was provided for `_PromotionCode`, the `INSERT` statement executes. If the value for `_PromotionCode` is NULL, the subsequent `INSERT` statement doesn't execute.

We end our conditional statement with `END` `IF`, and we end the stored procedure with `END`. Let's put everything together as a single statement that we can use to create our stored procedure and create it so we can test the conditional logic of our decision structure. We'll use the same `DELIMITER` commands in MySQL to change the delimiter from a semicolon so we can execute the entire stored procedure:

```sql
DELIMITER //

CREATE PROCEDURE AddPromotion (
    IN _PromotionID int,
    IN _PromotionCode varchar(10),
    IN _PromotionStartDate datetime,
    IN _PromotionEndDate datetime
 )
 BEGIN

IF _PromotionCode IS NOT NULL THEN
    INSERT INTO promotion (
    PromotionID,
    PromotionCode,
    PromotionStartDate,
    PromotionEndDate
    )
    SELECT
        _PromotionID,
        _PromotionCode,
        _PromotionStartDate,
        _PromotionEndDate
        ;
END IF;
END //

DELIMITER ;
```

If we execute this SQL, we should have our stored procedure AddPromotion ready to execute. First, let's try executing the code with the following arguments for the parameters:

```sql
CALL AddPromotion (14, '2OFF2023', '2023-01-04', '2023-02-11');
```

Executing this command should result in the message "1 row(s) affected" in the Output panel. We can verify that the row was inserted with the following query (results shown in figure 21.6):

```sql
SELECT *
FROM promotion
WHERE PromotionID = 14;
```

##### Figure 21.6 The results of selecting any rows from the promotion table in which the PromotionID is 14. We inserted that row with the AddPromotion stored procedure.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH21_F06_Iannucci.png)

Now that we've verified that our stored procedure works correctly when the result of our `IF` condition is true, let's try it when the result is false. We'll execute the following SQL, which uses an argument of NULL for the `_PromotionCode` parameter:

```sql
CALL AddPromotion (15, NULL, '2023-07-04', '2023-07-11');
```

Executing this command should result in the error message "0 row(s) affected" in the Output panel. We can use a similar query to the one we used to confirm that no row was inserted. Executing the following query returns no results:

```sql
SELECT *
FROM promotion
WHERE PromotionID = 15;
```

You've written the stored procedure AddPromotion, and you've examined the decision structure it uses to determine whether a row should be inserted into the promotion table. But what if you aren't familiar with the internal workings of this stored procedure? Suppose that you executed the `CALL` of the stored procedure that returned no results. Wouldn't you want to know why executing the stored procedure didn't work as expected?

When you're writing stored procedures, especially when you're using any kind of decision structure, you usually want the procedure to provide some sort of feedback on the results of the decisions made. To add functionality to your stored procedure that provides feedback or takes some other action if the condition of the evaluation is false, or even if you want to add more conditions, we can use some other keywords.

### 21.2.2 ELSE

Just as `IF` evaluates whether a condition is true, `ELSE` provides an alternative action to take if an evaluated condition is determined to be false or NULL. We can use `ELSE` the same way that we used `IF`, except that it comes *after* the `IF` statement.

##### Warning

Although the use of `ELSE` is optional, any statement with `ELSE` can follow only an `IF` statement. If you write an `ELSE` statement without a preceding `IF` statement, you'll get a syntax error.

Let's review the decision structure we want to have in AddPromotion:

1. If `_PromotionCode` is not NULL, we insert the values provided in the promotion table.
2. Otherwise, we don't want to insert the values; we want to return a message explaining why the insert was skipped.

Think of `ELSE` as a shorter version of the word *otherwise,* meaning that after the `IF` condition isn't met, this is the last thing we do. To accomplish the goal stated in point 2, we need to add this bit of SQL before the `END` `IF`:

```sql
ELSE
    SELECT 'No PromotionCode, INSERT skipped.' AS Message;
```

This code says that if a condition of false or NULL exists for the `IF` condition, we'll select a literal value of `No` `PromotionCode,` `INSERT` `skipped.` that will be returned as `Message`. It's about as straightforward as can be. Let's execute the following SQL to `DROP` `AddPromotion` and then re-create the AddPromotion stored procedure with our new logic:

```sql
DROP PROCEDURE AddPromotion;

DELIMITER //

CREATE PROCEDURE AddPromotion (
    IN _PromotionID int,
    IN _PromotionCode varchar(10),
    IN _PromotionStartDate datetime,
    IN _PromotionEndDate datetime
)
BEGIN

IF _PromotionCode IS NOT NULL THEN
    INSERT INTO promotion (
    PromotionID,
    PromotionCode,
    PromotionStartDate,
    PromotionEndDate
    )
    SELECT
        _PromotionID,
        _PromotionCode,
        _PromotionStartDate,
        _PromotionEndDate
        ;
ELSE
    SELECT 'No PromotionCode, INSERT skipped.' AS Message;
END IF;

END //

DELIMITER ;
```

After executing the preceding SQL, we can try executing the previous call of AddPromotion with an argument of NULL for `_PromotionCode`. The results in figure 21.7 show that we received the message included in the `ELSE` statement:

```sql
CALL AddPromotion (15, NULL, '2023-07-04', '2023-07-11');
```

##### Figure 21.7 The output from the `ELSE` statement we added to AddPromotion, which provides a helpful message about why an insert was skipped

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH21_F07_Iannucci.png)

So far, we've used a simple decision structure that evaluates one particular condition, but we can also evaluate multiple conditions using `IF` and `ELSE`.

### 21.2.3 Multiple conditions

Our previous decision structure evaluated the value for `_PromotionCode` for being `NOT NULL`, but if we want AddPromotion to be a useful stored procedure, we probably should account for possible null values in the other parameters as well. To do so, we have to change our decision structure to reflect several options for evaluation. Let's consider the new decision structure we want to have in AddPromotion:

1. If `_PromotionID` is null, we want to return a message explaining why the insert was skipped.
2. If `_PromotionCode` is null, we want to return a message explaining why the insert was skipped.
3. If `_PromotionStartDate` is null, we want to return a message explaining why the insert was skipped.
4. If `_PromotionEndDate` is null, we want to return a message explaining why the insert was skipped.
5. Otherwise, we want to insert the values provided in the promotion table.

The SQL inside AddPromotion that handles this decision structure needs a new keyword. This keyword is `ELSEIF`, which is an additional `IF` - to handle all these additional evaluations. We use it like this:

```sql
IF _PromotionID IS NULL THEN
    SELECT 'No PromotionID, INSERT skipped.' AS Message;
ELSEIF _PromotionCode IS NULL THEN
    SELECT 'No PromotionCode, INSERT skipped.' AS Message;
ELSEIF _PromotionStartDate IS NULL THEN
    SELECT 'No PromotionStartDate, INSERT skipped.' AS Message;
ELSEIF _PromotionEndDate IS NULL THEN
    SELECT 'No PromotionEndDate, INSERT skipped.' AS Message;
ELSE
    INSERT INTO promotion (
    PromotionID,
    PromotionCode,
    PromotionStartDate,
    PromotionEndDate
    )
    SELECT
        _PromotionID,
        _PromotionCode,
        _PromotionStartDate,
        _PromotionEndDate
        ;
END IF;
```

If we modify AddPromotion to include this logic using `ELSEIF`, we have a message for every possible reason for failure to insert the values.

##### Note

In SQL Server, the keyword `ELSEIF` is represented by two words: `ELSE` `IF`.

Unfortunately, we don't get any message if the query inserts the values successfully. To add that functionality, we need to add another statement to our `ELSE` statement. Because we now have multiple statements to execute after `ELSE`, we need to group those statements in a single block with the `BEGIN` and `END` keywords. Let's look at the `ELSE` statement to add the message `INSERT` `successful.`:

```sql
ELSE BEGIN
    INSERT INTO promotion (
    PromotionID,
    PromotionCode,
    PromotionStartDate,
    PromotionEndDate
    )
    SELECT
        _PromotionID,
        _PromotionCode,
        _PromotionStartDate,
        _PromotionEndDate
        ;

    SELECT 'INSERT successful.' AS Message;
    END;
```

The `BEGIN` and `END` keywords allow you to group multiple statements in a single block, much as your entire stored procedure has a `BEGIN` and `END`. As your SQL becomes more complex, you'll use these keywords frequently, especially in stored procedures.

##### Try it now

`DROP` and `CREATE` the AddPromotion stored procedure, using the decision structure changes discussed in section 21.2.3. Then try executing it with arguments of NULL for the various parameters and verifying that the correct message is returned.

You've made a lot of decisions today, so you have a wealth of options for making decisions in your queries. In chapter 22, you'll learn to use these decision-making options to evaluate individual rows of data.

## 21.3 Lab

1.  Using `CASE`, write a query that returns the following three columns from the promotion table:

•   The PromotionCode column

•   The first character of the PromotionCode column with the alias `PromotionCodeLeft1`

•   The sentence `This` `promotion` `is` `$X` `off.` with the alias `PromotionDiscount` and the literal value of X replaced by the value of the second column

2.  This query uses a `CASE` statement to attempt to replace NULL values for MiddleName with an empty string. Why doesn't it work as expected?

```sql
SELECT
    FirstName,
    CASE MiddleName
        WHEN NULL THEN ''
        ELSE MiddleName
        END AS MiddleName,
    LastName
FROM author;
```

3.  If you want to add logic to AddPromotion to skip an insert if the argument for `_PromotionCode` exists in the promotion table, what might that code look like? (This question is a bit tricky, but try to use what you've learned to answer it.)

## 21.4 Lab answers

1.  You can use the `LEFT` function (mentioned in chapter 14) to help with the query, which might look something like this:

```sql
SELECT
    PromotionCode,
    LEFT(PromotionCode, 1) AS PromotionCodeLeft1,
    CASE LEFT(PromotionCode, 1)
        WHEN 1 THEN 'This promotion is for $1 off.'
        WHEN 2 THEN 'This promotion is for $2 off.'
        WHEN 3 THEN 'This promotion is for $3 off.'
        END AS PromotionDiscount
FROM promotion;
```

2.  You cannot evaluate for equality with NULL because NULL can never equal NULL. To work as intended, the `CASE` statement has to evaluate an expression like this:

```sql
SELECT
    FirstName,
    CASE WHEN MiddleName IS NULL THEN ''
        ELSE MiddleName
        END AS MiddleName,
    LastName
FROM author;
```

3.  You might add this logic by using an additional `ELSEIF` statement like this:

```sql
ELSEIF NOT EXISTS (SELECT PromotionCode FROM promotion WHERE PromotionCode = _PromotionEndDate) THEN
    SELECT 'Duplicate PromotionCode, INSERT skipped.' AS Message;
```

This example is a little more advanced than the ones covered in this chapter, but it shows that you can do more than check for a NULL value in a Boolean expression. You can even use `EXISTS` and `NOT` `EXISTS` to evaluate entire queries.

