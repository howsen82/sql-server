# 14 Querying with Functions

Chapter 12 looked at a handful of functions—commands that perform some sort of predefined calculation. We looked specifically at basic aggregate functions that allow us to quickly calculate things like the sum of a range of values, as well as the minimum, maximum, and average values for a given range.

This chapter examines even more functions that open more possibilities in SQL, including those that allow us to select and filter specific string, date and time, and other informational values. First, though, we'll take a broader look at when we should and shouldn't use functions.

## 14.1 The problems with functions

Functions are incredibly useful for selecting specific parts of values, calculating values, and manipulating values in SQL. They're like magic spells we can perform by adding an extra word in our SQL. Functions, however, have two big problems that we need to discuss before we use them throughout our queries.

### 14.1.1 Function commands vary for each RDBMS

The core keywords and clauses we've used up to now are universal for the most part. When we write SQL using `SELECT`, `FROM`, `WHERE`, and `GROUP BY`, we know that the code will work not only in MySQL but also in any relational database management system (RDBMS) we use. Functions, however, are not universal, and many of the functions we examine in this chapter have some variation for one or more RDBMSes.

Now, that doesn't mean the functions you'll learn and practice are only for MySQL; many of them work in another RDBMS. But it does mean that if you try to use them in another RDBMS and encounter a syntax error, you'll likely need to do a little research to find out the correct keyword for that particular RDBMS. That said, I'll do my best to note these variations throughout this chapter because they can be obstacles to taking your new SQL skills to another RDBMS.

### 14.1.2 Function commands can be inefficient

I noted at the end of chapter 12 that the `DISTINCT` keyword has to do extra work by reading all the values in a range and returning the requested values without duplicates. I noted that it should be used very carefully with large sets of data because you don't want to use server resources unnecessarily.

Depending on their use, nearly all the functions discussed in this chapter also need to read all the values in a range. Although functions are incredibly useful and appear to be shortcuts to achieving a desired output, we need to be mindful of their use with large sets of data. Although using functions with a large data set can get us the correct answers, functions may use more resources and therefore may not be the most efficient way to use SQL.

Again, these warnings aren't meant to discourage you from using functions. You just need to be aware of their limitations and effects.

## 14.2 String functions

*String functions* allow us to select parts of string data, which we often have to do when we need to present data in a way that differs from how it's stored in the database. Examples of these situations are returning customer names in uppercase for a mailing list and eliminating unnecessary leading or trailing spaces from a value.

### 14.2.1 Case functions

The first string function we'll try is the `UPPER` function, which converts a string of characters to uppercase characters. We'll use it here to view only the customers in California, which are identified by the State value of CA and shown in figure 14.1. We'll also include the actual values stored for FirstName and LastName for comparison:

```sql
SELECT
    FirstName,
    LastName,
    UPPER(FirstName),
    UPPER(LastName)
FROM customer
WHERE State = 'CA';
```

##### Figure 14.1 The first and last names of all customers in California, first as they exist in the customer table and then in uppercase when we use the `UPPER` function

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH14_F01_Iannucci.png)

The syntax for `UPPER`, as for most functions, is to call the function and then contain the column name, variable, or other value where the function is applied inside parentheses. For this reason, you'll often see functions with parentheses in their names, such as `UPPER()`.

Although you may have observed that the third and fourth columns in figure 14.1 are uppercase characters, as expected, also notice the names of the columns returned. Those names are the columns as they appear in our `SELECT` clause, which will be the default name if one is not assigned. Let's run the query again, this time using column aliases of the prefix `Upper` with assigned column names (results shown in figure 14.2):

```sql
SELECT
    FirstName,
    LastName,
    UPPER(FirstName) AS UpperFirstName,
    UPPER(LastName) AS UpperLastName
FROM customer
WHERE State = 'CA';
```

##### Figure 14.2 The first and last names of all customers in California, first as they exist in the customer table and then as all uppercase when we use the `UPPER` function with defined column names

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH14_F02_Iannucci.png)

That result is more readable, and we could use similar logic with another function to make all the characters in the columns lowercase. There are few reasons to present values in lowercase, but if lowercase is required, this function is available. For this task, we can use `LOWER` instead of `UPPER` (results shown in figure 14.3) with corresponding column aliases:

```sql
SELECT
    FirstName,
    LastName,
    LOWER(FirstName) AS LowerFirstName,
    LOWER(LastName) AS LowerLastName
FROM customer
WHERE State = 'CA';
```

##### Figure 14.3 The first and last names of all customers in California, first as they exist in the customer table and then as all lowercase when we use the `LOWER` function with defined column names.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH14_F03_Iannucci.png)

### 14.2.2 Trim functions

The other thing we want to try is removing leading or trailing spaces. We have a few options for this task, with three separate functions: `RTRIM`, `LTRIM`, and `TRIM`. The `RTRIM` function removes all *trailing spaces* from a value—all spaces to the right of the last nonspace character. `LTRIM` removes all *leading spaces* from a value—all spaces to the left of the first nonspace character. `TRIM` is the same as applying both `LTRIM` and `RTRIM` to a value; it removes all leading and trailing spaces.

##### Note

You could use two functions on the same value, such as `SELECT (RTRIM(LTRIM(SomeValue))`. Make sure that your logical order is correct because the innermost function will be executed first. You may encounter SQL like this written by folks who lacked either the knowledge or ability to use the `TRIM` function to remove both leading and trailing spaces.

Let's try these functions using a variable with leading and trailing spaces. This example may seem nonsensical, but I assure you that if you ever need to program an interface for manual data entry, you'll have to deal with leading and trailing spaces in the data.

Our variable will have three leading spaces and two trailing spaces. We'll also assign column names in our query (trimmed results shown in figure 14.4):

```sql
SET @Word = '   word  ';
SELECT
    @Word AS WordAsEntered,
    LTRIM(@Word) AS WordLTRIM,
    RTRIM(@Word) AS WordRTRIM,
    TRIM(@Word) AS WordTRIM;
```

##### Figure 14.4 The results of selecting the character string `' word '`, which has three leading spaces and two trailing spaces, with the three trim-related functions. `LTRIM` removes the left leading spaces, `RTRIM` removes the right trailing spaces, and `TRIM` removes both leading and trailing spaces.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH14_F04_Iannucci.png)

Admittedly, we can't see the spaces in the results too well just by looking at them. But we can use another function to verify that leading and trailing spaces have been removed: `LENGTH`. This function returns the length in terms of the number of characters, including spaces, for each value.

We'll have to do a little math for the next example, but only basic addition and subtraction. The word *word* has four characters, and if we add three leading spaces and two trailing spaces, the length of our word as entered should be nine characters.

If we trim the three left (leading) spaces, our `LTRIM` value should be 6 (9 minus 3). If we trim the two right (trailing) spaces, our `RTRIM` value should be 7 (9 minus 2). Finally, if we trim all leading and trailing spaces, we should have a length of four characters for the word *word*.

Test this with SQL and the `LENGTH` function by wrapping it around the functions you used for trimming spaces. That's right—you can execute a function from inside another function. When you do this, though, remember that the innermost function always gets executed first. Figure 14.5 shows the results of this code:

```sql
SET @Word = '   word  ';
SELECT
    LENGTH(@Word) AS WordAsEnteredLength,
    LENGTH(LTRIM(@Word)) AS WordLTRIMLength,
    LENGTH(RTRIM(@Word)) AS WordRTRIMLength,
    LENGTH(TRIM(@Word)) AS WordTRIMLength;
```

##### Figure 14.5 The results of selecting the length of the strings with the `LENGTH` function after applying different trim functions to the string `'word'`. Because the removal of leading and trailing spaces can be difficult to see, we can use `LENGTH` to validate the results.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH14_F05_Iannucci.png)

Again, trimming data this way is important because you generally don't want to store or display data with leading spaces. Those spaces not only take up unnecessary space in your database but also can cause problems with common tasks such as filtering and sorting.

##### Note

SQL Server has no `LENGTH` function. Instead, use `LEN` to find the length of a string.

### 14.2.3 Other string functions

Although each RDBMS has its own set of string functions, some functions are so commonly used that they're available in nearly every RDBMS. Table 14.1 lists a few of those functions.

##### Table 14.1 Common string functions and definitions available in most RDBMSes [(view table figure)](https://drek4537l1klr.cloudfront.net/iannucci/HighResolutionFigures/table_14-1.png)

| Function name | Description |
| --- | --- |
| `LEFT` | Gets a specified number of leftmost characters from a string |
| `REPLACE` | Searches and replaces a substring of values in a string |
| `RIGHT` | Gets a specified number of rightmost characters from a string |
| `SUBSTRING` | Gets a substring starting from a specified position with a specific length |

## 14.3 Date and time functions

We can not only parse string values with functions but also do the same with date and time values. Most RDBMSes have appropriately named functions for determining the `YEAR`, `MONTH`, `DAY`, `HOUR`, `MINUTE`, and `SECOND`, which can be useful when we want to find information based on one or more parts of the date.

### 14.3.1 Date functions that return numeric values

Suppose that we want to find a list of all order IDs from 2015 like the one shown in figure 14.6. We could use the `YEAR` function in a query that checks the year of all orders in the orderheader table and returns the requested data. Let's include OrderID and OrderDate in our query:

```sql
SELECT
    OrderID,
    OrderDate
FROM orderheader
WHERE YEAR(OrderDate) = 2015;
```

##### Figure 14.6 The results of all OrderIDs and OrderDates from orders placed in 2015. We got these results by using the `YEAR` function.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH14_F06_Iannucci.png)

You could do the same thing with one of the other six functions related to date and time. I'm sure you're already thinking about how to use variables in this kind of search to add even more flexibility for filtering. Or maybe you're starting to consider that you could also use the functions to select parts of a date and time value.

Let's use all these functions on the OrderDate of the first order. That order has an OrderID of 1001 (results shown in figure 14.7):

```sql
SELECT
    OrderDate,
    YEAR(OrderDate),
    MONTH(OrderDate),
    DAY(OrderDate),
    HOUR(OrderDate),
    MINUTE(OrderDate),
    SECOND(OrderDate)
FROM orderheader
WHERE OrderID = 1001;
```

##### Figure 14.7 The results of all the date and time parts of the OrderDate of the first order in the orderheader table

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH14_F07_Iannucci.png)

These functions may not seem helpful now, but chapter 15 suggests some practical ways to use date and time parts to determine the results of various calculations. Also, these functions aren't the only date and time functions. Section 14.3.2 discusses two others that may interest you.

### 14.3.2 Date functions that return string values

The `DAYNAME` and `MONTHNAME` functions give you a little more information about date values by returning string values for the names of the month and day of a given date. Although the month may seem obvious if you know the numeric value, it's highly unlikely that you'll remember the name of the day of the week on which the order was placed. Modify the preceding query to use these functions so you can find out (results shown in figure 14.8):

```sql
SELECT
    OrderDate,
    YEAR(OrderDate),
    MONTHNAME(OrderDate),
    DAY(OrderDate),
    DAYNAME(OrderDate)
FROM orderheader
WHERE OrderID = 1001;
```

##### Figure 14.8 The results of using the `MONTHNAME` and `DAYNAME` to determine the names of the month and day of the first order in the orderheader table

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH14_F08_Iannucci.png)

##### TIP

Remember these functions when you have to compile reports that need a date time value formatted a certain way or the name of the month or day specified.

### 14.3.3 Other date and time functions

Table 14.2 lists some common date and time functions that are available in nearly every RDBMS.

##### Table 14.2 Common date and time functions available in most RDBMSes [(view table figure)](https://drek4537l1klr.cloudfront.net/iannucci/HighResolutionFigures/table_14-2.png)

| Function name | What It Gets |
| --- | --- |
| `DATE` | Only the date from a date and time value |
| `DAYOFWEEK` | The numeric day of the week for a date value |
| `DAYOFYEAR` | The numeric day of the year for a date value |
| `LAST_DAY` | The last date of the month for a date value |
| `QUARTER` | The quarter of the year for a date value |
| `TIME` | Only the time from a date and time value |
| `WEEKOFYEAR` | The numeric week of the year for a date value |

As you can imagine, these functions have many potential uses for finding information about date and time values stored in a database. But what if you need to find information about right now, such as the who, where, and when of a query? Fortunately, functions are available to answer these questions.

## 14.4 Informational functions

Each RDBMS has its own set of ways to answer the who, where, and when of a query, although some common functions are typically used. Let's start with the when, as in "When does a query occur?"

### 14.4.1 Date and time information

To determine when "right now" is, you commonly use the function `CURRENT_TIMESTAMP`. This function grabs the current time on the server where your database is located, which in this case is most likely the computer you're using for your local installation of the sqlnovel database. The format of the date and time will be similar to what you've seen for the date and time values so far, in the format `[year-month-day hour:minute:second]`.

Note that when you use `CURRENT_TIMESTAMP`, you still need to use parentheses as you would with any function, even though you don't pass a value:

```sql
SELECT CURRENT_TIMESTAMP() AS RightNow;
```

I'm not going to show you the results because my results will be different from yours, and your results will be different every time you execute this function.

##### Try it now

Determine the current time on your database server, using the `CURRENT_TIMESTAMP` function.

This function isn't the only one related to the current time. If you need only the day or the time, most RDBMSes have the `CURRENT_DATE` and `CURRENT_TIME` functions as well. You can try these functions with a query like this one:

```sql
SELECT
    CURRENT_DATE() AS CurrentDate,
    CURRENT_TIME() AS CurrentTime;
```

Because `CURRENT_TIMESTAMP` is a bit longer than most function names, many RDBMSes have another function that does the same thing with fewer letters. In MySQL, this function is `NOW`. You can use the following SQL to confirm this fact and see that both functions return the same value:

```sql
SELECT
    CURRENT_TIMESTAMP() AS RightNow,
    NOW() AS AlsoRightNow;
```

One last thing to note about these functions is that you can use them with other functions covered in this book. If you need to return the name of today's day of the week, for example, you could determine it with a query like this:

```sql
SELECT DAYNAME(NOW()) AS CurrentDayOfWeek;
```

### 14.4.2 Connection information

Now let's move on to a final set of functions that tell you who and where you are. Although you're working with only one database throughout this book, in professional experience, you're likely to connect to multiple databases at various times to query different data. If you're ever uncertain about which database you're connected to, you can identify it with the `DATABASE` function (result shown in figure 14.9):

```sql
SELECT DATABASE();
```

##### Figure 14.9 The results of selecting the current database used by the connection, which is the sqlnovel database

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH14_F09_Iannucci.png)

Also, you may use more than one login to connect to a database. This can happen when you change between your personal login to another used by a specific application or report system, often for testing. You can determine the username being used by your connection with the `CURRENT_USER` function:

```sql
SELECT CURRENT_USER();
```

##### Note

MySQL has both the `USER` and `CURRENT_USER` functions, both of which return information about the username. Most RDBMSes include `CURRENT_USER` but not `USER`.

Finally, as I noted in the installation directions in chapter 1, the MySQL database engine gets periodic updates that are represented in the version number. When you're connecting to a database, the version number is rarely obvious. If you want to see the current version number, use the `VERSION` function:

```sql
SELECT VERSION();
```

The version is returned as a string of three numbers separated by periods; the first number is the main version. For the exercises in this book, you should be using version 8 or later.

That should be enough new functions for now. In chapter 15, you'll discover even more functions that allow you to manipulate values and perform calculations in many practical ways.

## 14.5 Lab

1.  I noted that most functions take a parameter of some kind, but I didn't use any parameters with `CURRENT_TIMESTAMP`. What happens if you pass a value to `CURRENT_TIMESTAMP`, such as the number 2?

2.  Using the date functions discussed in this chapter, how can you determine a count of orders that were placed on a Monday?

3.  What two variables can you use to determine the longest title name in the title table? How can you write a query to determine this name?

## 14.6 Lab answers

1.  The `CURRENT_TIMESTAMP` function accepts integers as parameters; they determine the precision used in the date and time value returned. Adding `2` also return milliseconds of the date and time:

```sql
SELECT CURRENT_TIMESTAMP(2);
```

2.  You can use the `DAYNAME` function, which allows you to filter rows in the orderheader table with an order date on a Monday:

```sql
SELECT COUNT(OrderID) AS MondayOrders
FROM orderheader
WHERE DAYNAME(OrderDate) = 'Monday';
```

3.  You can use the `MAX` and `LENGTH` functions to determine the longest title name in the title table:

```sql
SELECT MAX(LENGTH(titlename))
FROM title;
```

The next part may be a bit more challenging because it involves a subquery. You can filter on titles with the length determined by the preceding query by using a subquery in the predicate to determine the name of the longest title with a length matching the `MAX` length:

```sql
SELECT TitleName
FROM title
WHERE LENGTH(TitleName) =
    (SELECT MAX(LENGTH(TitleName))
    FROM title);
```

