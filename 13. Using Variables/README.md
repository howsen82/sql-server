# 13 Using Variables

We've written and executed a lot of SQL queries so far, and a good number of those queries involved filtering the results on specific values. Through many examples, we've seen how to filter on a particular order or title ID, customer name, or date range, and every time, we've specified the literal value for filtering in our SQL. A *literal* value is specific, such as the number `4` or the date `2020-10-06`. Using literal values is helpful for learning and practice, but when you use SQL outside this book, you'll need to write more flexible queries.

If you want to look at the total sales of a title for a given month, such as March 2021, you can write a query to do that now. But what if you want to run a similar query for April or need total sales for a different title or a different range of dates? Do you have to write a different query for each title and date range?

I assure you that you don't. All you have to do is learn how to use variables. A *variable* is a memory-based object that stores a value that, once defined, can be used repeatedly throughout a query or in subsequent queries. More important, the stored value can vary from one execution to another, which means that the value is *variable*—hence, the name.

Considering the flexibility that variables provide, you'll use them with great frequency throughout your SQL. Let's get started!

## 13.1 User-defined variables

Although there are different kinds of variables, the ones we'll use in this chapter are known as *user-defined variables*. The name is self-explanatory because the user (you or I) will *define* these variables, which means assigning them a name and a value. All these variables start with the at sign (`@`), so when you see `@` in SQL, you're likely to be looking at a variable.

Before we use any variable, we must declare it. Since chapter 2, we've verbally declared our intentions in English to help us understand the syntax of queries, but sometimes, we also need to declare things in our SQL. Let's look at how to do that in MySQL.

### 13.1.1 Declaring your first user-defined variable

Declaring a variable typically requires two pieces of information to start: the *name* of the variable and the *value* of the variable. Suppose that we want to write a query to filter on title name. We could start with a sensible variable name like `@TitleName` and the value `'The` `Sum` `Also` `Rises'`. We could make a straightforward verbal declaration: "I would like to declare a variable named @TitleName, and I would like to assign it the value of The Sum Also Rises." The SQL used for this declaration is similar in logic, using the new keyword `SET`:

```sql
SET @TitleName = 'The Sum Also Rises';
```

As with many things in SQL, the syntax is similar to the order of our verbal declaration. We declare a variable by setting its name with `SET` and then assign a value using `=` and a literal value.

##### Note

When you use `SET`, you can use either `=` or `:=` as the assignment operator. Which one you use is a matter of personal preference, although as you'll see later in the chapter, in at least one instance, you must use `:=` for your variable declaration.

You may have noticed that we didn't specify what data type to use. That's because MySQL determines the data type based on the value we used. In our example, we used a character string (`'The` `Sum` `Also` `Rises'`) as the value of the variable, so our variable is a string data type.

Other permissible data types for variables include integer, decimal, and float, which are numeric data types. If the data for the variable doesn't fit one of the permissible types, the relational database management system (RDBMS) will convert the values to a permissible data type. Date and time values, which we've used throughout this book, are treated as strings.

##### Warning

This method of declaring variables in MySQL isn't universal. When you use a different RDBMS, such as SQL Server or PostgreSQL, you have to declare a user-defined variable using the `DECLARE` keyword and also assign it a specified data type.

If necessary, we can confirm the value of our variable at any time with a simple `SELECT` statement, as shown in figure 13.1:

```sql
SELECT @TitleName;
```

##### Figure 13.1 The results of selecting `@TitleName`, which shows the value of the variable. It also shows the variable name in the header.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH13_F01_Iannucci.png)

Although selecting a variable to confirm its value may seem trivial, you'll use this method quite a bit when you use variables. When you write a SQL query, this method is useful for checking the values of a variable periodically; it's also helpful for troubleshooting complex SQL scripts that aren't returning the desired values.

### 13.1.2 Understanding rules for user-defined variables

I should note a few rules about using variables before we go any further. There aren't many rules, and they aren't difficult to remember, but they're crucial to using variables correctly:

* *The first character of a variable name must be `@`*. Using `@` in the variable name tells the RDBMS you're working with a variable.
* *The remaining characters in a variable name must be alphabetic or numeric*. As you can see with the use of `@`, nonalphanumeric characters can have special meanings in SQL. Use only letters and numbers in your variable names.
* *Variable names can be no more than 64 characters.* You want to use descriptive variable names so that others who read your SQL can easily understand their purposes. But if the name of any of your variables is anywhere near 64 characters, you're probably being a bit too descriptive.
* *Variable names are not case-sensitive*. If you declare a variable named `@Variable`, any use of `@VARIABLE`, `@variable`, or `@VaRiAbLe` will refer to the same one.
* *A user-defined variable can hold only a single value.* You can't include multiple values, although you can change the value of a variable throughout your SQL if you so desire.
* *A user-defined variable exists only for the duration of the connection.* Databases and tables *persist*, which means that they exist until they're explicitly removed. Unlike those objects, variables don't persist, so when you close MySQL Workbench or any other application you use to connect to a database, any variables you've declared no longer exist.

### 13.1.3 Using your first user-defined variable

With all those rules out of the way, we're ready to put variables to use. Let's write some SQL to use a variable that helps us select the TitleID, TitleName, and PublicationDate from the title table, with the results shown in figure 13.2:

```sql
SET @TitleName = 'The Sum Also Rises';
SELECT
    TitleID,
    TitleName,
    PublicationDate
FROM title
WHERE TitleName = @TitleName;
```

##### Figure 13.2 The TitleID, TitleName, and PublicationDate from the title table for `'The` `Sum Also` `Rises'`, as filtered using a user-defined variable

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH13_F02_Iannucci.png)

Now we have this bit of SQL, which admittedly is rather short. Suppose that we had much more SQL to execute for a given title—perhaps to find the number of titles sold or the states of residence of customers who purchased the title. Whatever the case, if we set and filtered on a variable throughout the query as we did earlier, to query a different title, we'd have to make the change in only one part of our SQL.

Here's how that task looks in practice. Let's change the variable to `'Pride` `and Predicates'` and execute our query again. The following query produces the results shown in figure 13.3:

```sql
SET @TitleName = 'Pride and Predicates';
SELECT
    TitleID,
    TitleName,
    PublicationDate
FROM title
WHERE TitleName = @TitleName;
```

##### Figure 13.3 The TitleID, TitleName, and PublicationDate from the title table for `'Pride and Predicates'`, as filtered using a user-defined variable

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH13_F03_Iannucci.png)

Again, this bit of SQL is simple, but I hope it helps you see the power of using variables to make your script more flexible and reusable. Variables are used in nearly every programming language, and plenty of examples in this chapter and subsequent chapters show how to use them effectively.

##### Try it now

Declare a variable with a name of your choosing, and use it to select the TitleID, TitleName, and PublicationDate from the title table for any particular title.

## 13.2 Filtering with variables in FROM and HAVING clauses

Let's look at some practical ways to use variables in SQL. Suppose that we want the date of every order of any particular title. We can use a variable for this task, and in this case, we'll start with `'The Sum Also Rises'`. With the table-joining logic we've used before, we could write something like this (results shown in figure 13.4):

```sql
SET @TitleName = 'The Sum Also Rises';

SELECT
    oh.OrderDate
FROM orderheader oh
INNER JOIN orderitem oi
    ON oh.OrderID = oi.OrderID
INNER JOIN title t
    ON oi.TitleID = t.TitleID
WHERE t.TitleName = @TitleName;
```

##### Figure 13.4 The OrderDate for any order that included the title `'The` `Sum Also` `Rises'`

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH13_F04_Iannucci.png)

You can change that title name to any other valid title and return the corresponding set of order dates. As you might imagine, if your variable is set to a value that's not included in the title table, your result set would be zero rows.

Another common way to use variables is to find information about orders on a particular day, week, month, or year. Let's find the names of all customers who placed orders for any titles in November 2021, as shown in figure 13.5. In this case, we'll use two variables to represent the start and end dates of our range:

```sql
SET @DateStart = '2021-11-01',
    @DateEnd = '2021-11.30';

SELECT
    c.FirstName,
    c.LastName,
    oh.OrderDate
FROM customer c
INNER JOIN orderheader oh
    ON c.CustomerID = oh.CustomerID
WHERE oh.OrderDate BETWEEN @DateStart and @DateEnd;
```

##### Figure 13.5 The FirstName and LastName of any customer who placed an order in November 2021, along with the OrderDate

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH13_F05_Iannucci.png)

Interestingly, we can put this filter in a different part of the predicate. Instead of putting the filter in the `WHERE` clause, we can make it a condition in the `JOIN`. (This isn't typically how filtering is done in SQL, although you may notice it in other people's code.) Here's what the preceding query would look like if we filtered on our variables in the `FROM` clause as part of a `JOIN` condition:

```sql
SET
    @DateStart = '2021-11-01',
    @DateEnd = '2021-11-30';

SELECT
    c.FirstName,
    c.LastName,
    oh.OrderDate
FROM customer c
INNER JOIN orderheader oh
    ON c.CustomerID = oh.CustomerID
    AND oh.OrderDate BETWEEN @DateStart and @DateEnd;
```

We can also use a variable to see how many titles sold above a specific quantity. We can apply the aggregation techniques we learned in chapter 12 here, with a `HAVING` clause as a filter that uses a variable. Let's get a list of all the TitleNames that sold 10 or more copies, as shown in figure 13.6:

```sql
SET @MinimumQuantitySold = 10;

SELECT
    t.TitleName,
    SUM(oi.Quantity) AS TotalQuantitySold
FROM orderitem oi
INNER JOIN title t
    ON oi.TitleID = t.TitleID
GROUP BY t.TitleName
HAVING SUM(oi.Quantity) >= @MinimumQuantitySold;
```

##### Figure 13.6 The TitleName and TotalQuantitySold of all titles that sold at least 10 copies

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH13_F06_Iannucci.png)

The results show four titles that meet the threshold of at least 10 copies (Quantity) sold. If we want to change the threshold of our filter to another value, all we need to do is change the value of the `@MinimumQuantitySold` variable.

## 13.3 Assigning an unknown value to a variable

One useful aspect of variables allows us to create and use a variable even when we don't know the explicit value on which we want to filter. If I asked you the value of TitleID for TitleName The Sum Also Rises, would you know it? Honestly, I wrote everything in this database, and even I can't recall that value.

Although we may not have memorized the TitleID values, we've used them countless times in our queries because these values constitute the relationship between the orderheader and title tables.

### 13.3.1 Reviewing how a query works

Let's take a moment to consider how TitleID is used in the first query in section 13.2 and how we can use one variable to collect the value for another variable. Here's that query, which looks for the order dates of the title The Sum Also Rises:

```sql
SET @TitleName = 'The Sum Also Rises';

SELECT
    oh.OrderDate
FROM orderheader oh
INNER JOIN orderitem oi
    ON oh.OrderID = oi.OrderID
INNER JOIN title t
    ON oi.TitleID = t.TitleID
WHERE t.TitleName = @TitleName;
```

Let's walk through the joins in this table, starting at the bottom and working our way up. Why do it this way? Your RDBMS will probably start finding your results by filtering rows in the title table. Filtering typically means fewer rows to read, and fewer rows to read means fewer rows to join with other tables, which is more efficient than reading all the rows in all the tables, joining them, and then applying filtering.

In this query, we used the variable `@TitleName` to find any rows in the title table that matched our query, which happens to be one row. Then we join that row to any related rows to orderitem via the matching TitleID values, and we join to any related rows in orderheader using the matching OrderID values in both tables. When we have related values through all tables, we can select the values for OrderDate to determine when the titles were ordered.

### 13.3.2 Assigning an unknown variable with SELECT

This query is a relatively simple one with a couple of joins. It's important to note, though, that joins require the RDBMS to do extra work because it has to read and relate the data in different tables. Fortunately, we can reduce the number of joins in our query in section 13.3.1 by creating a variable for the TitleID values because we know that only one value in the title table matches the TitleName value for The Sum Also Rises.

We can find the value for TitleID, which is unknown, by declaring our variable a bit differently. We're going to use a `SELECT` statement instead of `SET` because we're selecting a value from an existing table to use in our variable.

First, though, let's take a step back. Suppose that we want to return the value of TitleID instead of using it to assign a value to our `@TitleID` variable. We might write our SQL this way, using a `@TitleName` variable for the name of the title:

```sql
SET @TitleName = 'The Sum Also Rises';

SELECT TitleID
FROM title
WHERE TitleName = @TitleName;
```

##### Try it now

I keep talking about this value, so execute this query to find the TitleID value for The Sum Also Rises.

We can use the logic from this query to assign the value of this TitleID to a new `@TitleID` variable. Figure 13.7 shows the output for the assigned value:

```sql
SET @TitleName = 'The Sum Also Rises';

SELECT @TitleID := TitleID
FROM title
WHERE TitleName = @TitleName;
```

##### Figure 13.7 Selecting an unknown value for TitleID in `@TitleID` will result in output that shows the value that was selected—in this case, 108.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH13_F07_Iannucci.png)

The `FROM` and `WHERE` clauses are identical to those in the preceding query, but the `SELECT` clause looks unlike anything we've done so far. With the logic of `SELECT` `@TitleId` `:=` `TitleID`, we can do two things at the same time with our `SELECT` clause: select the TitleID value and assign that value to our `@TitleID` variable.

Also notice that we used the `:=` operator instead of `=` in our SQL. Earlier in this chapter, I mentioned that you can use `=` or `:=` when you assign a value to a variable using `SET`. When you assign a value to a variable using `SELECT` in MySQL, however, you can use only the `:=` operator.

##### Warning

As I noted earlier for the `SET` keyword, this method of assigning an unknown value to a variable is different in nearly every RDBMS. Although the process is fundamentally similar, it's important to know the correct syntax for your RDBMS.

One other interesting side effect of assigning a value to a variable with `SELECT` is the fact that the results are output to the Results panel. `SELECT` statements result in output in MySQL, and in this case, the result shows the value that was assigned to the variable, with the SQL used in the `SELECT` statement as the column header.

### 13.3.3 Considering performance with variables

Now that we've learned how to assign a value to a variable using `SELECT`, let's see how this looks with the overall query that was intended to find the order dates for The Sum Also Rises (results shown in figure 13.8):

```sql
SET @TitleName = 'The Sum Also Rises';

SELECT @TitleID := TitleID
FROM title
WHERE TitleName = @TitleName;

SELECT
    oh.OrderDate
FROM orderheader oh
INNER JOIN orderitem oi
    ON oh.OrderID = oi.OrderID
WHERE oi.TitleID = @TitleID;
```

##### Figure 13.8 The OrderDate for any order that includes the title The Sum Also Rises. This time, we used the `SELECT` keyword instead of `SET` keyword to get the results shown in figure 13.4.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH13_F04_Iannucci.png)

This latest query uses two variables but requires only one join in the final statement instead of two. Although this query is still relatively trivial in terms of work required of the RDBMS, in queries against larger sets of data, reducing joins as we've done here can offer significant improvements in performance.

##### Note

If you look closely at the Results panel, you'll see two sets of results, with two separate tabs at the bottom of the panel. Those shown in figure 13.8 are the results you wrote your SQL to determine, but as you may have guessed, the results in the "hidden" tab are the output from the first `SELECT` where the variable value was assigned.

### 13.3.4 Troubleshooting considerations with variables

Consider a request to find the title, quantity, and price of the first order in a particular year, such as 2021. To do this, first we need to find the date of the first order in 2021; then, using that value, we find the information about the order placed on that date. Our database contains so few orders that there are no more than one per day, which makes the task simpler. Let's start by determining the date of the first order in 2021, as shown in figure 13.9.

##### Figure 13.9 The OrderDate of the first order placed in 2021, which is shown only because of the `SELECT` statement. We'll use this value later to determine more information about the order placed on that day.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH13_F09_Iannucci.png)

I haven't covered this topic yet, but we could use the `SET` method to assign an unknown value to our variables instead of `SELECT`. To do this, however, we'd have to use a kind of subquery, like this:

```sql
SET @FirstOrderDate = (
    SELECT MIN(OrderDate)
    FROM orderheader
    WHERE OrderDate BETWEEN '2021-01-01' AND '2021-12-31');

SELECT @FirstOrderDate AS FirstOrderDate;
```

This query produces the correct result, but we can see the value assigned to the variable only if we explicitly use a separate `SELECT` statement. As we saw earlier, using `SELECT` instead of `SET` to assign this value also shows us the value assigned to the variable (figure 13.10):

```sql
SELECT @FirstOrderDate := MIN(OrderDate)
FROM orderheader
WHERE OrderDate BETWEEN '2021-01-01' AND '2021-12-31';
```

##### Figure 13.10 The OrderDate of the first order placed in 2021, shown without a second `SELECT` statement. The SQL used in the `SELECT` statement is the column header.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH13_F10_Iannucci.png)

Whichever method you use is a matter of preference. This is largely determined by whether you want the output to show whether the correct value is being used because `SET` doesn't show the value of a variable by default, as `SELECT` does. As you write a query, it may be helpful to use the `SELECT` method to verify that the values assigned to your variables are correct to prevent incorrect results. Seeing the values in the Results panel may give you more confidence in the effectiveness of your SQL.

For now, let's use the method with `SELECT` to determine the first order of 2021 and to select the title, quantity, and price of that order. The following query returns the results shown in figure 13.11:

```sql
SELECT @FirstOrderDate := MIN(OrderDate)
FROM orderheader
WHERE OrderDate BETWEEN '2021-01-01' AND '2021-12-31';

SELECT
    t.TitleName,
    oi.Quantity,
    oi.ItemPrice
FROM orderheader oh
INNER JOIN orderitem oi
    ON oh.OrderID = oi.OrderID
INNER JOIN title t
    ON oi.TitleID = t.TitleID
WHERE oh.OrderDate = @FirstOrderDate;
```

##### Figure 13.11 The TitleName, Quantity, and ItemPrice of the items in the first order placed in 2021

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH13_F11_Iannucci.png)

It's good to have options for using variables in your SQL. Now you should have a better understanding of the pros and cons of using `SET` or `SELECT` to assign value to your user-defined variables.

## 13.4 Other notes about variables

Before we get to the lab exercises, we have a few more points about variables to consider.

### 13.4.1 Assigning a literal value using SELECT

Although I didn't cover this topic, we can use `SELECT` to assign a literal value instead of `SET`. The choice is mostly a matter of preference, but we've seen throughout the chapter that `SELECT` offers more options for working with `FROM`, `WHERE`, and other clauses. We'd use this syntax to assign a date value, for example:

```sql
SELECT @SomeDate := '2021-11-30';
```

### 13.4.2 Assigning a value of NULL to a variable

Often, we want to start with a variable with a null value assigned to it and see later in our SQL whether it gets another value assigned. The variable type doesn't matter until a value gets assigned, in which case the variable type will change to the data type of the value. We can assign a null value to a variable with `SET` or `SELECT`, using either of the following lines of SQL:

```sql
SET @NullVariableWithSET = NULL;

SELECT @NullVariableWithSELECT;
```

### 13.4.3 Changing the type of data used by a variable

In MySQL, variables can have different values assigned to them throughout your SQL. As I just noted, the variable data type can change if you start with `NULL` but later assign a string, integer, or other kind of value. There aren't many use cases for changing the data type of a variable, but if you want to, you can even assign different data types to a variable throughout your SQL. In the following example, the first assigned value is a number, and a string data type is assigned later:

```sql
SET @SomeVariable = 1;

SELECT @SomeVariable AS FirstValue;

SET @SomeVariable = 'The Sum Also Rises';

SELECT @SomeVariable AS SecondValue;
```

Although it's possible to change a variable type throughout your SQL, doing so falls in the category "Things You Can Do but Shouldn't." I note this option only in case you make a mistake and accidentally reuse a variable more than once; you won't get an error message or warning that you've done so.

## 13.5 Lab

1.  In this chapter, I noted that in MySQL you must use `:=` when assigning a value using `SELECT`. What happens if you use `=` instead?

2.  I also noted that you can assign only a single value to a variable. What happens if you execute the following query? Is a value assigned to the variable, and if so, what is the value?

```sql
SELECT @TitleID := TitleID
FROM title;
```

3.  Review the final query in section 13.3.4, and update it, using variables for the start date and end date.

4.  Write a query to find total sales dollars (in terms of Quantity times Price) for any customer, with the customer's FirstName and LastName as variables.

## 13.6 Lab answers

1.  If you use `=` instead of `:=` to assign values to a variable in a `SELECT` statement, the value of the variable will be null. The MySQL RDBMS uses `=` to test for equality, as we've seen numerous times when using filters and joins. In the `SELECT` statement, it determines that the two values are not equal because the variable has no value. Remember: null is the absence of data.

2.  The value is 108, although if you execute the query, you see all the values for TitleID in the results. Because only one value can be assigned to the variable, the final value is assigned. For this reason, be careful to select only a single value when using this method to assign values to a variable.

3.  You could use SQL this way to add more flexibility to this query, allowing for easy changes at the top to examine different ranges of data:

```sql
SET
    @DateStart = '2021-01-01',
    @DateEnd = '2021-12-31';

SELECT @FirstOrderDate := MIN(OrderDate)
FROM orderheader
WHERE OrderDate BETWEEN @DateStart and @DateEnd;

SELECT
    t.TitleName,
    oi.Quantity,
    oi.ItemPrice
FROM orderheader oh
INNER JOIN orderitem oi
    ON oh.OrderID = oi.OrderID
INNER JOIN title t
    ON oi.TitleID = t.TitleID
WHERE oh.OrderDate = @FirstOrderDate;
```

4.  You have a few ways to do this. Here's one way:

```sql
SET @FirstName = 'Chris';
SET @LastName = 'Dixon';

SELECT @CustomerID := CustomerID
FROM Customer
WHERE FirstName = @FirstName
    AND LastName = @LastName;

SELECT
    @FirstName AS FirstName,
    @LastName AS LastName,
    SUM(oi.Quantity * oi.ItemPrice) AS TotalSalesDollars
FROM orderheader oh
INNER JOIN orderitem oi
    ON oh.OrderID = oi.OrderID
WHERE oh.CustomerID = @CustomerID;
```

