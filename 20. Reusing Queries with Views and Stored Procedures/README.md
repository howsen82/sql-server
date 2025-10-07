# 20 Reusing Queries with Views and Stored Procedures

Through 19 chapters, we've written a lot of SQL queries. We've used filters, functions, aggregations, and more to find specific data. We've even added, updated, and removed data, and we've used variables to enable our scripts to do the same things over and over with different values.

In this chapter, we'll bring a lot of that work together by moving from executing SQL scripts to saving scripts as objects in the database—scripts that anyone who has the necessary permissions can execute. Depending on the relational database management system (RDBMS) we're using, we can use a few objects to store these scripts. For now, we'll focus on two nearly universal objects: views and stored procedures.

A *view* stores a `SELECT` statement and provides a single result set that can be used like a table. A *stored procedure* stores one or more queries that can be executed at the same time to perform nearly any required task in a database.

## 20.1 Views

*Views* are database objects we create based on a `SELECT` statement. Views provide a single result set that resembles a table, which is why they're often referred to as *virtual tables*. The term *virtual tables* indicates that we can use views like tables in our queries.

Referring to views as *virtual tables* is a bit misleading, though, because views aren't tables and don't contain data. It may be more helpful to think of them as `SELECT` statements with a name, although that description doesn't fully describe their usefulness.

Views allow us to reuse a query easily; they also enable us to reduce a complex query to a simple, accessible object. We can take that object and assign users permissions to enable them to use the view (or not).

### 20.1.1 Creating views

Any view starts with a `SELECT` statement. If we want to create a view that shows the names of titles and their category names, we could write a query like this (results shown in figure 20.1):

```sql
SELECT
    t.TitleName,
    c.CategoryName
 FROM title t
INNER JOIN category c
    ON t.CategoryID = c.CategoryID;
```

##### Figure 20.1 All TitleName values from the title table and the related CategoryName values in the category table

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH20_F01_Iannucci.png)

To create a view with this query, we need to create a SQL statement in the following order:

1. `CREATE` `VIEW`
2. View name
3. `AS`
4. `SELECT` statement

Using this easy syntax, we can create a view named vw_TitleCategory like this:

```sql
CREATE VIEW vw_TitleCategory
AS
SELECT
    t.TitleName,
    c.CategoryName
FROM title t
INNER JOIN category c
    ON t.CategoryID = c.CategoryID;
```

When we create the view, it saves our `SELECT` statement to be executed whenever we want. To see the results of our view, we select it from the view as though it were a table:

```sql
SELECT *
FROM vw_TitleCategory;
```

The results of executing the preceding query, shown in figure 20.1, are the same as those of executing our original SQL statement.

For queries that you have to write over and over, views can save you coding time because the desired result set is ready to execute. Again, this view doesn't contain data; it simply calls the data via the underlying SQL statement when it's executed. But executing isn't all we can do with views.

### 20.1.2 Filtering with views

Just as we can filter the results of a table, we can filter the results in a `WHERE` clause, as we've done many times in other queries. If we want to see only the titles in our view that are in the Mystery category, as shown in figure 20.2, we modify our `SELECT` from the view to have the appropriate filtering in a `WHERE` clause, like this:

```sql
SELECT *
FROM vw_TitleCategory
WHERE CategoryName = 'Mystery';
```

##### Figure 20.2 The results of selecting all rows from vw_TitleCategory with a CategoryName value of Mystery

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH20_F02_Iannucci.png)

With views, we can do nearly everything we've done with tables. We can filter results, order results, and aggregate data. We can even join views to other tables and views, but to do that with our view, we have to make some changes.

### 20.1.3 Joining views

As you may recall from the many times you've joined tables, you must define relationships to make joins successful. The vw_TitleCategory view contains two columns, but neither is related to any key values that form relationships with other tables. No tables in the database have any relationship that uses the TitleName or CategoryName column as a key value.

To use our view with other tables, we need to add the key values from the underlying tables. For our view, that means adding the TitleID from the title table and the CategoryID from the category table. Now, we could add the CategoryID from the title table instead of from the category table, but it's good practice to use primary keys instead of foreign keys in views whenever possible.

##### Warning

Just as column names in tables must be unique, column names in views must be unique as well. If you attempt to create a view with column names that aren't unique, most RDBMSes, including MySQL, return an error message telling you that duplicate column names exist.

We'll modify our view using syntax similar to what we used to create our view. But we'll use the `ALTER` keyword instead of `CREATE`, as we did when we modified a table in chapter 18:

* `ALTER` `VIEW`
* View name
* `AS`
* Modified SQL query

Using this syntax, we can modify our vw_TitleCategory view to include the two additional columns:

```sql
ALTER VIEW vw_TitleCategory
AS
SELECT
    t.TitleID,
    t.TitleName,
    c.CategoryID,
    c.CategoryName
 FROM title t
INNER JOIN category c
    ON t.CategoryID = c.CategoryID;
```

##### Try it now

If you haven't created and altered the vw_TitleCategory yet, modify the preceding query from `ALTER` `VIEW` to `CREATE` `VIEW` to create it. You'll use it again later in this chapter.

After executing our `ALTER` `VIEW` statement, we can examine the results of our view with the following query (results shown in figure 20.3):

```sql
SELECT *
FROM vw_TitleCategory;
```

##### Figure 20.3 The results of selecting all rows and columns from vw_TtileCategoryID, which now includes the TitleID and CategoryID columns

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH20_F03_Iannucci.png)

Notice that the results aren't in any particular order. This is fine because we don't want to order the values in our view unless we have to. As with a `SELECT` statement, adding an `ORDER` `BY` clause to a view can make query performance much worse when we're dealing with millions of rows of data.

Now we can not only join this view to other tables and view, but also create calculated columns. If we want to see how many titles were sold in each category, for example, we can join our view in a query to the orderitem table by using the relationship of the TitleID columns. Here's what that query looks like (results shown in figure 20.4):

```sql
SELECT
    tc.CategoryName,
    SUM(oi.Quantity) AS TitlesOrdered
FROM vw_TitleCategory tc
LEFT OUTER JOIN orderitem oi
    ON tc.TitleID = oi.TitleID
GROUP BY tc.CategoryName;
```

##### Figure 20.4 The results of the number of titles ordered in each category

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH20_F04_Iannucci.png)

### 20.1.4 Considerations for views

Views are incredibly useful, but some rules and caveats apply. Here are some of the main factors you should consider when you create and use views:

* A view can't have the same name as any other view or table. Keep this rule in mind when naming your views.
* When you name views, try to use a naming convention that identifies them as views, not tables. Our example uses the prefix vw_ in the view name. A consistent naming convention becomes important when other users use views you created to look at queries; you want them to be able to distinguish tables from views easily.
* It's a good idea to add columns for the primary key and foreign key values to the `SELECT` clause of the SQL statement used by the view. This approach allows you to relate the results of your view to other tables and views.
* Chapter 11 examined using *subqueries*—queries contained in other queries—to find the data you want with SQL. Similarly, views can call other views via subqueries or joins. These views used within other views are referred to as *nested views*. Avoid using them, though, because they dramatically degrade query performance.
* If your view contains a calculated column, as the preceding query does, always create an alias for the column name. Most RDBMSes require every column in a view to have a defined name.
* Although it may seem improbable, many RDBMSes allow data to be updated or even inserted into views. Doing so can be problematic because these changes can affect data in multiple tables, so in general, you should avoid this practice.

##### NOTE

In light of that last point, views aren't the right tool for modifying data in SQL, but stored procedures can be wonderful tools for that purpose. They're also a better tool for selecting data.

## 20.2 Stored procedures

Like views, stored procedures allow us to store a SQL statement in our database so we can easily reuse it. We can also assign users permissions to determine whether they can use the stored procedures. Unlike views, stored procedures allow for even more complexity, such as executing multiple queries and passing values through variables.

### 20.2.1 Creating stored procedures

Let's start by turning our SQL statement from section 20.1.1 into a stored procedure. Nearly every RDBMS has a basic syntax for creating stored procedures, which looks like this:

1. `CREATE` `PROCEDURE`
2. Name of the stored procedure
3. SQL for the stored procedure to execute

##### Note

Unfortunately, each RDBMS has its own subtle syntax differences for handling the creation of a stored procedure. But don't let that fact keep you from learning about them, because despite these differences, the use of stored procedures is similar across RDBMSes except SQLite, which doesn't support stored procedures.

To create our stored procedure in MySQL, first we must change the delimiter. When we wrote our first query in chapter 2, we learned to add a semicolon (a *statement terminator*) to the end of a query to tell the RDBMS where the SQL in our query stops. MySQL is rigid about the statement terminator, which can be a concern when writing stored procedures. Because these procedures can contain multiple statements, the first semicolon our code encounters would look like the end of the stored procedure to the RDBMS.

To work around this situation, we'll briefly change the statement terminator to a value other than a semicolon. In our SQL, we'll change the statement terminator to double slashes (`//`) by using the MySQL-specific keyword `DELIMITER`, create our stored procedure with the standard semicolon delimiter, and then use `DELIMITER` to change the statement terminator back to a semicolon. We'll name the procedure GetTitleCategory. Here's the SQL to create our stored procedure that gets all TitleName values and the associated CategoryName values, followed by notes that explain what the code does:

```sql
DELIMITER //                                    #1

CREATE PROCEDURE GetTitleCategory()             #2
BEGIN                                           #3
SELECT                                          #4
    t.TitleName,
    c.CategoryName
 FROM title t
INNER JOIN category c
    ON t.CategoryID = c.CategoryID;
END //                                          #5

DELIMITER ;                                     #6
```

Now that we have created our stored procedure, we can put it to use. To execute our stored procedure, we'll use the `CALL` keyword (results shown in figure 20.5):

```sql
CALL GetTitleCategory;
```

##### Figure 20.5 The results of all TitleName values from the title table and their related CategoryName values in the category table, as returned by the stored procedure GetTitleCategory

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH20_F05_Iannucci.png)

This stored procedure is a basic one. We can do much more with stored procedures, so in section 20.2.2, we'll add functionality to pass a variable and filter our results.

##### Note

The writing of a stored procedure is specific to an RDBMS, and so is the execution. Although MySQL, PostgreSQL, and MariaDB use the `CALL` keyword, SQL Server and Oracle use `EXEC`.

### 20.2.2 Using variables with stored procedures

One of the biggest advantages of stored procedures over views is the fact that stored procedures have parameters. A *parameter* is a variable that can be passed into or out of a stored procedure. Any stored procedure can have multiple parameters that we can use for anything from filtering data to changing values in tables to determining the output results of a stored procedure. When we use parameters with a stored procedure, each parameter must have three properties defined:

* The name
* The data type
* Whether the parameter is used for input or output

The final property gives us choices on how to use a parameter. If a parameter is defined for *input*, a value is passed into the stored procedure for use. If a parameter is defined for *output*, the value of the parameter is determined during the execution of the stored procedure and returned.

##### Note

MySQL offers a third option for parameters: `INOUT`. This option allows a parameter to be passed in, modified if necessary, and then passed back out. This option isn't available in every RDBMS.

We can modify our GetTitleCategory stored procedure to have an input parameter that filters on TitleName. To modify a stored procedure in MySQL, however, first we have to drop it, similar to the way we dropped tables in chapter 18:

```sql
DROP PROCEDURE GetTitleCategory;
```

Now we can re-create our stored procedure with an input parameter. We'll name our parameter `_TitleName` so it won't be confused with the column named TitleName, and we'll define the data type as the one defined for the TitleName column in the title table. We can see the data type of columns for any table in MySQL by using `SHOW COLUMNS`. This statement is how we'd use `SHOW` `COLUMNS` to find the data types of the columns in the title table (results shown in figure 20.6):

```sql
SHOW COLUMNS FROM title;
```

##### Figure 20.6 The data types of all columns in the title table, returned by `SHOW` `COLUMNS`

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH20_F06_Iannucci.png)

We can see that the data type for the TitleName column is `varchar(50)`, so we'll define that data type for our input parameter. The last thing we have to do is use the parameter for filtering within the stored procedure. To accomplish this task, we'll add `WHERE t.TitleName` `=` `_TitleName`. The following SQL creates our new stored procedure:

```sql
DROP PROCEDURE GetTitleCategory;

DELIMITER //

CREATE PROCEDURE GetTitleCategory(
    IN _TitleName varchar(50)
    )
BEGIN
SELECT
    t.TitleName,
    c.CategoryName
 FROM title t
INNER JOIN category c
    ON t.CategoryID = c.CategoryID
WHERE t.TitleName = _TitleName;
END //

DELIMITER ;
```

Now we can declare a variable and pass it to the stored procedure to return only the results for the desired TitleName value. Using the value The Sum Also Rises, we can execute the stored procedure with the following SQL (results shown in figure 20.7):

```sql
SET @TitleName = 'The Sum Also Rises';
CALL GetTitleCategory (@TitleName);
```

##### Figure 20.7 The results of executing GetTitleCategory using the value The Sum Also Rises with the input parameter `_TitleName`

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH20_F07_Iannucci.png)

Now that we've added the input parameter `_TitleName` to GetTitleCategory, every execution of the stored procedure will require a value for that parameter. If we try to execute GetTitleCategory without a value for `_TitleName`, we'll get the error message "Incorrect number of arguments." In the context of a stored procedure, an *argument* is the value being passed to the parameter. Now GetTitleCategory expects a value for every execution, and if we don't pass one, we're passing zero arguments. As the error correctly calculates, zero is the incorrect number when one argument is expected.

When writing a stored procedure, we may want to account for the fact that we may not have a value for an argument, so instead of not passing an argument, we can pass one that has NULL as the value. This approach isn't uncommon. As we've seen throughout this book, NULL can be a value that has to be accounted for.

One way to handle passing a value of NULL to the `_TitleName` parameter is to return all rows in a result set, which we can do by using the `COALESCE` function (see chapter 15). We can change the filtering in our stored procedure to `WHERE` `t.TitleName` `= COALESCE(_TitleName,` `t.TitleName)` to account for a value of NULL used as an argument for the `_TitleName` parameter.

With this logic, we can execute our stored procedure with an argument of NULL. If a specific value is passed, our result set is still filtered on that value, but if NULL is passed, we return every row in which TitleName equals itself, which is every row.

Let's drop our stored procedure and re-create it with the new filtering logic that uses `COALESCE`. We can do all of this at the same time with the following SQL:

```sql
DROP PROCEDURE GetTitleCategory;

DELIMITER //

CREATE PROCEDURE GetTitleCategory(
IN _TitleName varchar(50)
    )
BEGIN
SELECT
    t.TitleName,
    c.CategoryName
FROM title t
INNER JOIN category c
    ON t.CategoryID = c.CategoryID
WHERE t.TitleName = COALESCE(_TitleName, t.TitleName);
END //

DELIMITER ;
```

With this new change, we can execute our stored procedure with or without a NULL value as an argument. Using a value of NULL returns values for all TitleNames, as shown in figure 20.8:

```sql
SET @TitleName = NULL;
CALL GetTitleCategory (@TitleName);
```

##### Figure 20.8 The results of all TitleName values from the title table and the related CategoryName values in the category table, returned by the stored procedure GetTitleCategory with an argument of NULL for the `_TitleName` parameter

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH20_F08_Iannucci.png)

If we execute GetTitleName with an argument that isn't NULL, like The Sum Also Rises, we get the filtered results for only that TitleName, as shown in figure 20.9:

```sql
SET @TitleName = 'The Sum Also Rises';
CALL GetTitleCategory (@TitleName);
```

##### Figure 20.9 The results of TitleName values from the title table and the related CategoryName values in the category table, returned by the stored procedure GetTitleCategory with an argument of The Sum Also Rises for the `_TitleName` parameter

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH20_F09_Iannucci.png)

##### Try it now

Create the final version of the GetTitleCategory stored procedure, and try executing it with different values as arguments for `_TitleName`, including NULL.

### 20.2.3 Considerations for stored procedures

I hope you see why stored procedures are popular ways to store our SQL statements in an RDBMS. Before you run out and change all your queries to stored procedures, however, you have a few significant factors to consider:

* Stored procedures can call other stored procedures and even pass variable values back and forth. Be careful about nesting stored procedures, which can become a headache to troubleshoot.
* As you do for views and other objects, use a consistent naming convention for your stored procedures so they're clearly identified as stored procedures and so other people will understand their purposes.
* If you're writing a stored procedure with parameters, always verify that the data types of the parameters match the data types of any columns they'll be evaluated against.
* If you're using variables to pass values to the parameters of a stored procedure, make sure that the data type of your variables matches that of the stored procedure.
* Because stored procedures can contain multiple queries, use lots of clear, descriptive comments in complex stored procedures to indicate what each part of your stored procedure is meant to do.

## 20.3 Differences between views and stored procedures

In this chapter, we've looked at two of the most popular ways to store SQL for reuse. As we've seen, views and stored procedures have different attributes. Let's review the main differences to help you better decide when to use either of them, as shown in table 20.1.

##### Table 20.1 Some main differences between views and stored procedures [(view table figure)](https://drek4537l1klr.cloudfront.net/iannucci/HighResolutionFigures/table_20-1.png)

| Attributes | View | Stored procedure |
| --- | --- | --- |
| Input | Doesn't use parameters | Can use parameters |
| Output | Can return only a single result set | Can return zero, one, or multiple result sets or output parameters |
| Multiple queries | Can contain only a single query | Can contain multiple queries |
| Relationships | Can be joined to other views or tables via relationships | Can't be joined to other objects |
| Dependencies | Can contain a query using tables or views but not stored procedures | Can contain queries using tables, views, or other stored procedures |

Although this chapter covers most of what you need to know about views, it only scratches the surface of the capabilities of stored procedures. As you'll see in chapter 21, you can create stored procedures to read data and write data, with logic determined by various conditions.

## 20.4 Lab

1.  Create a view named vw_Order that contains all the columns from the orderheader and orderitem tables except the OrderID column from the orderitem table. Recall that these tables are related by their OrderID columns.

2.  Why do you think you should exclude the OrderID column from the orderitem table?

3.  Create a stored procedure named GetOrder with the following specifications:

•   Uses the vw_Order view you just created

•   Has a parameter named `_OrderID` and filters the results based on matching the value of that parameter to the OrderID column of vw_Order

•   Joins the title table using the relationship of the TitleID columns

•   Returns the following columns: OrderID, OrderDate, TitleName, Quantity, and ItemPrice

4.  What kind of data type did you define for the `_OrderID` parameter in GetOrder, and why?

5.  After you create GetOrder, what happens if you execute the following code?

```sql
CALL GetOrder(1049)
```

## 20.5 Lab answers

1.  The SQL for your view should look something like this:

```sql
CREATE VIEW vw_Order
AS
SELECT
    oh.OrderID,
    oh.CustomerID,
    oh.PromotionID,
    oh.OrderDate,
    oi.OrderItem,
    oi.TitleID,
    oi.Quantity,
    oi.ItemPrice
 FROM orderheader oh
INNER JOIN orderitem oi
    ON oh.OrderID = oi.OrderID;
```

2.  If you hadn't excluded the OrderID column from the orderitem table in your view, there'd be two columns named OrderID. If you tried to create this view with two OrderID columns, the RDBMS would return an error because it wouldn't know which column to use.

3.  The SQL to create the GetOrder stored procedure should look something like this:

```sql
DELIMITER //

CREATE PROCEDURE GetOrder(
    IN _OrderID int
    )
BEGIN
SELECT
    o.OrderID,
    o.OrderDate,
    t.TitleName,
    o.Quantity,
    o.ItemPrice
FROM vw_Order o
INNER JOIN title t
    ON o.TitleID = t.TitleID
WHERE o.OrderID = _OrderID;
END //

DELIMITER ;
```

4.  You should have used an integer`(int)` data type for the `_OrderID` parameter because that's the data type of the OrderID column you'll be evaluating with the parameter. If you didn't know the data type, you could have discovered it using the following query in MySQL:

```sql
SHOW COLUMNS FROM orderheader;
```

5.  The command should execute, returning the results shown in figure 20.10. Executing the stored procedure this way shows that you can use either variables or literal values, such as 1049, when passing arguments to a parameter.

##### Figure 20.10 The results of executing your new GetOrder stored procedure with an argument of the literal value 1049 passed to the `_OrderID` parameter

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH20_F10_Iannucci.png)

