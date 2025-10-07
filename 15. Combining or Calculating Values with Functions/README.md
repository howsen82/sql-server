# 15 Combining or Calculating Values with Functions

Chapter 14 looked at several functions that allow us to return parts of data. This chapter looks at a few more functions that let you combine values in different ways and even perform calculations. Depending on the nature of the data you're working with, I'm sure you'll find some functions in this chapter that you'll use frequently. If you work with address data, for example, how can you combine all the columns for street, city, and more into a single column? Or if you work with financial reports, how can you make all your calculations show the desired precision of currency?

This chapter looks at these scenarios and more. We'll start with using functions to combine values.

## 15.1 Combining string values

I haven't discussed this topic yet, but you can use SQL to perform basic calculations, such as addition. Here's an example of basic addition (result shown in figure 15.1):

```sql
SELECT 1 + 1;
```

##### Figure 15.1 The results of calculating 1+1 using SQL

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH15_F01_Iannucci.png)

Being able to perform addition is useful if you're working exclusively with numeric data, but what if you need to combine string values? Unfortunately, as you can see in figure 15.2, using the plus sign (`+`) doesn't allow you to combine string values to get a desired result:

```sql
SELECT 'I' + ' ' + 'love' + ' ' + 'books!';
```

##### Figure 15.2 The results of combining string values with the plus operator, which doesn't combine all the values into a string

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH15_F02_Iannucci.png)

Instead of getting the result "I love books!," we get a result of 0. This result indicates that the calculation couldn't be completed because MySQL doesn't know how to "add" words together mathematically.

##### Note

You can use the plus sign to combine strings in SQL Server, but this approach won't work in most relational database management systems (RDBMSes).

A specific verb describes what we're trying to do here by combining two or more string values to form a single value. That verb is *concatenate*, and it's important to know because the function we'll use to concatenate our string values is `CONCAT`.

### 15.1.1 CONCAT

If we want to concatenate string values, we can use the `CONCAT` function to combine multiple values by specifying the list of values separated with commas. For the preceding query, we'd use this function as follows and get the result shown in figure 15.3:

```sql
SELECT CONCAT('I', ' ', 'love', ' ', 'books!');
```

##### Figure 15.3 The results of using the `CONCAT` function to create a single string value from multiple string values to form the "I love books!" output

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH15_F03_Iannucci.png)

This example may seem a bit silly, but as you'll soon see, this function is powerful. String values don't exist only in literal values like the ones we used in that query, of course. They also exist in the columns of tables or in variables. We can combine any of these string values with `CONCAT` just as easily. Let's create a variable for a title review and concatenate it with all the title names in the title table (results shown in figure 15.4):

```sql
SET @Review = ' is a great book!';
SELECT CONCAT(TitleName, @Review) AS TitleReview
FROM title;
```

##### Figure 15.4 The results of concatenating the values of the TitleName column in the title table with a string variable to form a single column of output

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH15_F04_Iannucci.png)

We can even use `CONCAT` with numeric or date data types. When we do so, however, we need to be aware that all values will be converted to string data types to concatenate values with different data types. This operation sometimes causes unexpected results in the sorting of concatenations that involve numeric or date values.

To demonstrate, let's combine the Price and TitleName values in the title table as shown in figure 15.5. We'll separate the values with spaces for readability. We want to sort the output from lowest values to highest, so we'll specify ascending order with `ASC` for emphasis:

```sql
SELECT CONCAT(Price, ' ', TitleName) AS PriceAndTitle
FROM title
ORDER BY PriceAndTitle ASC;
```

##### Figure 15.5 The results of the concatenated values of Price and TitleName with a space to separate them, sorted in ascending order of the concatenated value

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH15_F05_Iannucci.png)

What happened here? Numerically, the values 10.95 and 12.95 should be at the end of ascending order, but here, they appear at the beginning. This occurs because these numeric values had to be converted to string values for the concatenation, and they're sorted by the order of the characters. In this case, the first character in these concatenated values is 1, which ordinally comes before the first characters of the other values, which are 7, 8, and 9.

We can get the desired sorting in our output by using `ORDER BY` with the specific column we want to sort on, which in this case is Price. Even though the Price column by itself isn't in the result set, we can use it when sorting data, as the resulting rows show (figure 15.6):

```sql
SELECT CONCAT(Price, ' ', TitleName) AS PriceAndTitle
FROM title
ORDER BY Price;
```

##### Figure 15.6 The results of the concatenated values of Price and TitleName with a space to separate them, sorted in ascending order of Price

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH15_F06_Iannucci.png)

In case you've forgotten, I talked about this concept in chapter 4. To reiterate, just because a column isn't in the output doesn't mean you can't sort by that column in the `ORDER` `BY` clause. You can sort by any value or combination of values in the table that's in the `FROM` clause, provided that you aren't aggregating with a `GROUP` `BY` clause. This means your concatenated values can be ordered not only by price or title but also by title ID or publication date.

##### Try it now

Use the preceding query to select Price and TitleName as a concatenated value, but add `'$'` before the Price value to indicate the type of currency. If you're still sorting by Price rather than by the concatenated value, the order should still be in the expected ascending value. You can try sorting by TitleName or another column in the title table to change the order of the results.

You may not need to concatenate values of different data types often, as you just did, but if you work with customer data, you may need to produce output that concatenates first and last names in a single column. This output could be used in mailing lists, form emails, name badges, and so on.

As you might guess, concatenating first and last names is easy to do with `CONCAT`. Let's concatenate the values for FirstName and LastName in the author table, separated with a space and aliased as `AuthorName` (results shown in figure 15.7):

```sql
SELECT CONCAT(FirstName, ' ', LastName) AS AuthorName
FROM author;
```

##### Figure 15.7 The results of the FirstName and LastName columns of the author table concatenated into a single value and separated with a space

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH15_F07_Iannucci.png)

### 15.1.2 CONCAT_WS

As useful as the `CONCAT` function is, if we need to concatenate several values using the same separator, there may be an even better function. Although not every RDMBS supports it, most of them include a `CONCAT_WS` function to make concatenation with a separator a bit easier. The `CONCAT_WS` function is similar to `CONCAT`, with the exception that the first value provided is the separator used between all other values. We could produce the results shown in figure 15.7 with the following query, which uses the `CONCAT_WS` function with a space as the first value like this:

```sql
SELECT CONCAT_WS(' ', FirstName, LastName) AS AuthorName
FROM author;
```

Using `CONCAT_WS` doesn't make this particular SQL query any shorter. But if we had to separate more than two columns with a space or some other separator, the `CONCAT_WS` function is preferable to `CONCAT` because we need to specify the separator only once.

Because there happens to be a MiddleName column in the author table, let's try using `CONCAT_WS` to add it as well and concatenate each author's full name (results shown in figure 15.8):

```sql
SELECT CONCAT_WS(' ', FirstName, MiddleName, LastName) AS AuthorName
FROM author;
```

##### Figure 15.8 The results of using `CONCAT_WS` to concatenate the FirstName, MiddleName, and LastName of all rows in the author table

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH15_F08_Iannucci.png)

The `CONCAT_WS` function provides easy concatenation of all author names, which is remarkable if you consider that some of the middle names have null values. `CONCAT_WS` automatically accounts for those nulls and replaces them with empty strings when concatenating values, which `CONCAT` typically doesn't do.

With `CONCAT`, the nulls are not converted to empty strings, which can be problematic. As discussed in chapter 7, null values represent the absence of data, so any time we concatenate another value that isn't null to a null value, the result is always null.

Let's look at this concept in practice. If we attempt to produce the same results as those shown in figure 15.8 by using `CONCAT`, we'll be disappointed by the results. As figure 15.9 shows, any row with a null value for MiddleName will return a result of NULL:

```sql
SELECT CONCAT(FirstName, ' ', MiddleName, ' ', LastName) AS AuthorName
FROM author;
```

##### Figure 15.9 The results of using `CONCAT` to concatenate the FirstName, MiddleName, and LastName of all rows in the author table. The rows with NULL results are caused by null values in MiddleName.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH15_F09_Iannucci.png)

### 15.1.3 COALESCE

If we must account for possible null values when concatenating with the `CONCAT` function or any other function that doesn't change null values, the SQL language offers an additional function we can use. The `COALESCE` function is supported by every RDBMS and can be used to handle null values in concatenation functions. `COALESCE` takes a list of values that are provided to the function (similarly to how we provided a list of values to `CONCAT`) and returns the first non-null value from the list.

For the example shown in figure 15.10, we'll use `COALESCE` with the value for MiddleName as the first value in the list and then an empty string for the second value. Because we know that the empty string isn't null, we can trust that `COALESCE` will return non-null values for MiddleName or an empty string for null values.

We'll replace the MiddleName selection in the preceding query with the `COALESCE` function as described to ensure that there are no null values in the results (figure 15.10):

```sql
SELECT CONCAT(
    FirstName, ' ', COALESCE(MiddleName, ''), ' ', LastName
    ) AS AuthorName
FROM author;
```

##### Figure 15.10 The results of using `CONCAT` to concatenate the FirstName, MiddleName, and LastName of all rows in the author table. The `COALESCE` function replaced the null values for MiddleName with empty strings.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH15_F10_Iannucci.png)

Although `COALESCE` solved the problems created by the null values for MiddleName, if you look closely, you may see that in those rows, we now have two spaces between first and last names. Nothing is inherently wrong with that result, but in this case, using `CONCAT_WS` returned better-formatted results.

If you work with an RDBMS that doesn't offer `CONCAT_WS`, and you need to concatenate values like these and change the double spaces to single spaces, you can. You just need to use another common function that can convert a string of some values to another value.

## 15.2 Converting values

Now that we've combined multiple values to create a new value, let's look at a few more functions we can use to change values.

### 15.2.1 REPLACE

We can use the `REPLACE` function to change the occurrence of any combination of one or more characters within a string to some other combination. This combination is known as a *substring* because it is part of the overall string being evaluated.

The `REPLACE` function takes three values as input, in this order: the string you're going to search, the substring you want to replace, and the substring that will be the replacement.

Here's a simple example. If we wanted to change the way that the American word *check* is displayed to the British version, *cheque,* we could replace the letters *ck* in the word *check* with *que*, as shown in figure 15.11. We could write the following query using the `REPLACE` function:

```sql
SELECT REPLACE('check', 'ck', 'que');
```

##### Figure 15.11 The results of replacing `ck` in the string `'check'` with `que` to convert the word from the American English to British English

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH15_F11_Iannucci.png)

Returning to our problem of changing two spaces to one with our concatenated names in section 15.1.3, we can add a `REPLACE` function to our query to replace any occurrence of a double space with a single space (results shown in figure 15.12):

```sql
SELECT REPLACE(
    CONCAT(
        FirstName, ' ', COALESCE(MiddleName, ''), ' ', LastName
    )
    , '  ', ' ') AS AuthorName
FROM author;
```

##### Figure 15.12 The results of the concatenated author names, replacing the null values with an empty string using `COALESCE` and the resulting double spaces with a single space using `REPLACE`

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH15_F12_Iannucci.png)

To solve this particular problem, you used three functions. As you grow in your experience using SQL, this strategy won't be uncommon; every function performs a specific task, and you may have to think of ways like this to use more than one function in a query creatively.

### 15.2.2 CONVERT and CAST

Two functions are commonly used to convert values from one data type to another: `CAST` and `CONVERT`. Both functions convert a value from one data type to a different specified data type. We saw in chapter 13 that MySQL has some built-in functionality to convert values to different data types automatically, but not every RDBMS has this functionality. For many RDBMSes, you need to use one of these two functions to handle any type of data conversion. Also, although many RDBMSes offer both of these functions, some offer only one or the other. Fortunately, MySQL supports both, so you can practice some queries for each function.

Here's a simple example. The current date values are stored with both date and time. The time portion of this data isn't helpful in our sqlnovel database because we haven't captured any data for hours, minutes, or seconds. All we have are zeros for all the time values of PublicationDate. If we want to display only the date part of PublicationDate in the title table, we need to use one of these functions.

Let's look first at `CONVERT`, which requires two values: the value we're changing and then the desired data type to which we want to convert the data. In this example, we want to convert the data type from `DATETIME`, as it is stored in the title table, to `DATE`. Let's select both the original PublicationDate and our converted value to see the difference (results shown in figure 15.13):

```sql
SELECT
    PublicationDate,
    CONVERT(PublicationDate, DATE) AS PublicationDateNoTime
FROM title;
```

##### Figure 15.13 The results of selecting PublicationDate from the title table and converting it to remove the time portion of the value

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH15_F13_Iannucci.png)

The data types to which we can change our values vary by RDBMS. But be aware that converting string values such as names to a numeric or date value usually won't give you a useful result and may even return an error.

##### Try it now

Use the SQL in this section to convert the PublicationDate of the title table to `DATE` value; then try converting the TitleName to a `DATE` value as well. In MySQL, you should get a null value for the converted TitleName values because a string can't be converted to a valid date.

The `CAST` function is similar to `CONVERT` but has slightly different syntax. Instead of passing two separate values to the function, we pass a kind of phrase that uses the word `AS` instead of a comma. Here's how we'd use `CAST` for the preceding example:

```sql
SELECT
    PublicationDate,
    CAST(PublicationDate AS DATE) AS PublicationDateNoTime
FROM title;
```

Executing this query returns the same results as those shown in figure 15.13.

Why would you use one function instead of the other? Aside from personal preferences, you're likely to prefer `CAST` because it's considered to be a standard function of SQL; `CONVERT` is not. For this reason, you're much more likely to see `CAST` included in the list of functions for a given RDBMS, which means that SQL written for one RDBMS that includes `CAST` is more likely to be usable in another RDBMS.

##### Note

Although these two functions effectively do the same thing, the `CONVERT` function often has additional functionality that allows you to pass a third value for formatting the output. Check the documentation of your RDBMS to see whether `CONVERT` is supported and has additional options.

## 15.3 Numeric calculations with functions

In chapter 12, we discovered aggregate functions such as `MIN`, `MAX`, `AVG`, and `SUM`. These aggregate functions are types of numeric functions, which can be applied to numeric values for various calculations.

Many other numeric functions are available, but most of them perform specific mathematical calculations, such as the square root of a value, the logarithm, or the tangent. For now, let's focus on one mathematical function that most users can employ in some practical cases.

The `ROUND` function provides an easy way to solve common problems in which a value needs to be rounded with fewer decimal places—to convert decimal values of currency to integer values for simplicity, for example. Most businesses wouldn't publicly proclaim, for example, that they sold $1,000,000.32 in merchandise. Instead, they'd round the number to $1,000,000.

Although the sqlnovel database doesn't contain $1 million in sales, we can still use `ROUND` to produce the total sales in whole dollars for a given year without cents (fractions of a dollar). Let's see how we'd use the `ROUND` function to calculate the integer value for total sales value for all orders. Chapter 12 made a calculation on this very value by using the `SUM` function. Here, we'll add a second column that wraps around that calculation, using the `ROUND` function. Let's select both the sum and the rounded sum to show the difference (results shown in figure 15.14):

```sql
SELECT
    SUM(Quantity * ItemPrice) AS TotalOrderValue,
    ROUND(SUM(Quantity * ItemPrice)) AS TotalOrderValueRounded
FROM orderitem;
```

##### Figure 15.14 The results of the total value of all sales in dollars and cents, along with the same value rounded to an integer

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH15_F14_Iannucci.png)

Note that the rounding in this case made the value increase because anything equal to or greater than .50 is rounded to a higher value, whereas any value less than .50 is rounded to a lower value. We can easily prove this with a simple `SELECT` statement that rounds the value 573.49 to a lower value.

##### Try it now

Execute `SELECT` `ROUND(573.49);` to verify that this value will be rounded down to 573.

One other thing to note about the `ROUND` function: it has two parameters. The first parameter is for the number value, which we've used in this chapter. The second parameter, however, is optional; it specifies the number of places beyond the decimal point to which the number should be rounded. If a value for the second parameter isn't passed, it converts to an integer value (0 spaces beyond the decimal).

If you work with calculations involving currency, you'll likely need to use `ROUND` with both parameters. Suppose that you need to calculate the added sales tax for a purchase of a title that costs $9.95. If the tax is 5%, you could calculate that amount by multiplying $9.95 by .05. If you do, however, the resulting value for the tax value will be more than two decimal places. Because customers can't be charged fractions of cents, you can pass a value of 2 to the second parameter of `ROUND` to make the tax value match the currency. Use the following query to validate these results, showing both the calculated tax and the rounded tax (figure 15.15):

```sql
SELECT
    9.95 * .05 AS CalculatedTax,
    ROUND(9.95 * .05, 2) AS CalculatedTaxRounded
```

##### Figure 15.15 The results of using `ROUND` to round 573.49 from two decimal places to one, which rounds the value slightly higher

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH15_F15_Iannucci.png)

If you're required to work with more complicated calculations, your RDBMS likely has dozens of additional functions for mathematical equations. Table 15.1 lists some common mathematical functions that are available in nearly every RDBMS.

##### Table 15.1 Common mathematical functions available in most RDBMSes [(view table figure)](https://drek4537l1klr.cloudfront.net/iannucci/HighResolutionFigures/table_15-1.png)

| Function name | What it produces |
| --- | --- |
| `ABS` | The absolute value of a number |
| `CEIL` | The smallest integer not less than a number (rounding up) |
| `FLOOR` | The largest integer not more than a number (rounding down) |
| `MOD` | The remainder (modulo) of a number divided by another |
| `SQRT` | The square root of a number |

##### Note

In SQL Server, the `CEIL` function is replaced by `CEILING`.

So far, we've spent every chapter looking at ways to use SQL to select data from tables in a database and present the resulting output in different ways. In chapter 16, we'll start to look at ways to change data in tables through data manipulation.

## 15.4 Lab

1.  In the author table, select a single column aliased as `AuthorName` for all author first and last names, but in the format LastName, FirstName (such as Iannucci, Jeff).

2.  Write a query for which the output is a sentence that declares the PublicationDate of the first title. The output can be something like "The first title was published on 2001-01-30" except that you use the PublicationDate formatted with the date, not the time.

3.  It's common practice to ignore articles like the word *The* when sorting titles alphabetically. Write a query that returns the TitleName of all titles in the title table sorted alphabetically in this manner.

## 15.5 Lab answers

1.  You can use the `CONCAT_WS` function to format the names with a query like this:

```sql
SELECT CONCAT_WS(', ', LastName, FirstName) AS AuthorName
FROM author;
```

2.  Depending on which function you prefer to use, you can achieve this output in one of several ways. If you use `CAST`, your query might look like this:

```sql
SELECT CONCAT(
    'The first title was published on ', CAST(PublicationDate AS DATE), '.'
    ) AS FirstPublicationDate
FROM title
WHERE TitleID = 101;
```

You could get the same output with `CONVERT`:

```sql
SELECT CONCAT(
    'The first title was published on ', CONVERT(PublicationDate, DATE), '.'
    ) AS FirstPublicationDate
FROM title
WHERE TitleID = 101;
```

You could also use the `DATE` function, discussed in chapter 14:

```sql
SELECT CONCAT(
    'The first title was published on ', DATE(PublicationDate), '.'
    ) AS FirstPublicationDate
FROM title
WHERE TitleID = 101;
```

3.  This one may be a bit tricky because it requires using the `REPLACE` function in the `ORDER` `BY` clause, which you haven't done yet. By doing this, you can replace the word *The* in the TitleName with an empty string for sorting. Be sure to include the space after the word *The* to achieve the correct sort order:

```sql
SELECT TitleName
FROM title
ORDER BY REPLACE(TitleName, 'The ', '');
```

