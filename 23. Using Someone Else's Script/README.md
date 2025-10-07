# 23 Using Someone Else's Script

I hope you're feeling confident about all the SQL you've learned in this book. I've covered the most basic and frequently used keywords and statements, so you should be well prepared to fulfill requests to retrieve and even manipulate data in a relational database.

As with a foreign language, though, you need to be able to listen and read as well as speak or write. You need to be able to read existing SQL in stored procedures and elsewhere in whatever databases your organization has. Because this book is an introduction to the SQL language, you'll likely even find yourself looking on the internet for examples of SQL scripts that use keywords and concepts you haven't been exposed to yet.

To practice these vital skills and apply what you've learned, in this chapter, you'll review SQL examples written by someone else. Know that the examples will work, but you have to look at them closely to determine what the author intended. Also, these scripts go against the best practices you've learned, so you'll also be considering how to improve the SQL in the scripts.

There's no lab section at the end of the chapter because this chapter is like one large review. There aren't even any "Try it now" sidebars, but you're welcome to try these scripts if you want. Your main task here is to walk through the scripts, understand what they're meant to do, and improve them based on everything you've learned in this book.

## 23.1 Someone else's script: Creating a table

All these examples involve a new table named authorpayment that tracks royalty payments to authors. The rows in the table reflect the amount paid to each author by title and year. The presumption is that authors will be paid annually based on the sales of the titles.

### 23.1.1 The CREATE TABLE script

Let's start with the first script, which creates the table:

```sql
CREATE TABLE authorpayment (
    ID int,
    Author int,
    Title int,
    PaymentYear char(4),
    PaymentAmount decimal(7,2)
    );
```

Although this script isn't particularly verbose, I'm sure that if you recall the concepts and examples discussed in chapters 18 and 19, you'll immediately spot a few things that could be corrected. Take a moment to consider the script and make some notes about what you'd change. When you're ready, continue reading, and I'll share my thoughts.

### 23.1.2 Reviewing the CREATE TABLE script

The first thing you may notice is the column named ID, which is ambiguous. Unfortunately, naming the first column in a table ID is common when people design database tables in a hurry and without clear intentions. You always want to be clear about a column's purpose in case the column is used in other queries, so you should rename this column AuthorPaymentID.

Also, if this AuthorPaymentID column is intended to contain unique values that form the primary key of the table, you should add a `PRIMARY` `KEY` constraint to the column and even consider adding an `AUTO_INCREMENT` property to populate the column with values that increment automatically.

Next, the Author column appears not to have been fully thought out. It has the same data type as the AuthorID column used in several tables in the sqlnovel database, so for consistency, you should change the name to AuthorID. For the sake of data integrity, the column should also contain a foreign key reference to the author table so that AuthorID in the authorpayment table is populated only with values from the author table. Further, because every payment has to go to an author, you want to put a `NOT NULL` constraint on this column.

All these same points apply to the Title column. You should rename it TitleID for consistency, create a `FOREIGN` `KEY` constraint that references the TitleID values in the title table, and add a `NOT` `NULL` constraint to force a TitleID to be included in every row.

The PaymentYear column is a little odd because it has a data type of `char(4)`. This data type means that the values will be stored as a string of characters, even though years are numeric values. You don't need to worry about any non-numeric characters occurring in years when the authors will be paid, so you should use an integer (`int`) data type instead.

##### Note

You may want to use a character data type to store numeric values in one case: when you have leading zeros. U.S. zip codes used for mailing addresses are good examples because many zip codes start with at least one zero. If you entered the zip code 03872 into a column with an integer data type, it would be stored as 3872. For this reason, U.S. zip codes are typically stored as character (`char`) data types.

You should also place a `NOT` `NULL` constraint on the PaymentYear column because every row needs to reflect a particular year. Also, although it's not necessary to do so, you might want to put a `CHECK` constraint on the PaymentYear column to allow only values that fall within a certain range of years. This constraint would limit the data that could be entered, preventing many potential typos that could affect the integrity of the data. A good range for this constraint would be 2000 to 2100 because frankly, if this database is still being used in 2100, it will be well past time to upgrade.

Finally, the PaymentAmount column looks good because the data type, `decimal(7,2)`, means that you can accommodate payment values up to 99,999.99. You want to place a `NOT` `NULL` constraint on this column as well because every payment requires an amount. The only other addition you might consider is a `CHECK` constraint on the values to ensure that you have only positive values in this column; presumably, authors won't be paid negative amounts.

### 23.1.3 Improving the CREATE TABLE script

If you put everything together, the SQL to create the authorpayment table looks something like this:

```sql
CREATE TABLE authorpayment (
    AuthorPaymentID int NOT NULL AUTO_INCREMENT,
    AuthorID int NOT NULL,
    TitleID int NOT NULL,
    PaymentYear int NOT NULL CHECK (PaymentYear BETWEEN 2000 AND 2100),
    PaymentAmount decimal(7,2) NOT NULL CHECK (PaymentAmount BETWEEN 0.00 AND 99999.99),
    CONSTRAINT PK_AuthorPayment PRIMARY KEY (AuthorPaymentID),
    CONSTRAINT FK_authorpayment_author FOREIGN KEY (AuthorID) REFERENCES author(AuthorID),
    CONSTRAINT FK_authorpayment_title FOREIGN KEY (TitleID) REFERENCES title(TitleID)
);
```

I hope that you see and understand how these changes will help enforce data integrity and make the table understandable and consistent with the other tables in the database. In the future, you might even consider adding a unique index to cover AuthorID, TitleID, and PaymentYear because it appears that these values should be unique for each row. In addition, you might consider changing the primary key to use that combination of columns instead of the AuthorPaymentID, which would mean you wouldn't need to create the unique index.

## 23.2 Someone else's script: Inserting data

Next, let's look at a script that inserts rows into this new table. This stored procedure should run for each year, collecting sales information and then determining the royalty payment to the author.

### 23.2.1 The INSERT stored procedure

Here's a stored procedure for you to review:

```sql
DELIMITER //

CREATE PROCEDURE InsertAnnualPayment(
    IN _PaymentYear int
    )
BEGIN

DECLARE _Done boolean DEFAULT FALSE;
DECLARE _TitleID int;
DECLARE _AuthorID int;
DECLARE _Royalty decimal(5,2);
DECLARE _AuthorCount int;
DECLARE _TotalSales decimal(7,2);
DECLARE _PaymentAmount decimal(7,2);

DECLARE AllTitles CURSOR FOR

SELECT TitleID, AuthorID
FROM titleauthor
ORDER BY
    TitleID,
    AuthorOrder;

DECLARE CONTINUE HANDLER FOR NOT FOUND SET _Done = TRUE;

OPEN AllTitles;

GetTitles: LOOP

    FETCH AllTitles INTO _TitleID, _AuthorID;

    SET _Royalty = (
        SELECT Royalty
        FROM title
        WHERE TitleID = _TitleID
        );

    SET _AuthorCount = (
        SELECT COUNT(AuthorID)
        FROM titleauthor
        WHERE TitleID = _TitleID
        );

    SET _TotalSales = (
        SELECT SUM(orderitem.Quantity * orderitem.ItemPrice)
        FROM orderheader
        INNER JOIN orderitem
            ON orderheader.OrderID = orderitem.OrderID
        WHERE orderitem.TitleID = _TitleID
            AND YEAR(orderheader.OrderDate) = _PaymentYear
        );

    SET _PaymentAmount =
        COALESCE(CONVERT(
            ((_TotalSales * (_Royalty/100))/_AuthorCount), decimal(7,2))
            , 0.00);

    IF _PaymentAmount > 0.00 THEN
        INSERT authorpayment (
            AuthorID,
            TitleID,
            PaymentYear,
            PaymentAmount
            )
        SELECT
            _AuthorID,
            _TitleID,
            _PaymentYear,
            _PaymentAmount;
        END IF;

        IF _Done = TRUE THEN

            LEAVE GetTitles;
        END IF;

    END LOOP GetTitles;

    CLOSE AllTitles;

END //

DELIMITER ;
```

Take a moment to review the stored procedure and maybe even take some notes before proceeding.

### 23.2.2 Reviewing the INSERT stored procedure

If you read chapter 22, the first thing to notice about this stored procedure is that it uses a cursor to determine the annual royalty payments. I hope that when you see this cursor, you start wondering whether you can turn this row-by-row evaluation into a set-based evaluation instead.

To determine whether you can replace the cursor, first look at what the stored procedure is doing with the cursor. Let's go through each section of the stored procedure.

The start of the stored procedure is fairly standard. It looks as though an `int` value is required for the input parameter `_PaymentYear`, which is the only parameter:

```sql
DELIMITER //

CREATE PROCEDURE InsertAnnualPayment(
    IN _PaymentYear int
    )
BEGIN
```

After that section, several variables are declared. Although all the variables seem to have sensible data types, on closer inspection, you see that the `_Royalty` value doesn't match the `decimal(5,2)` data type used for the Royalty column in the title table. Mismatched data types can lead to data errors or inconsistencies. Also, as you go through the cursor, you'll see that many, if not all, of these variables may be unnecessary:

```sql
DECLARE _Done boolean DEFAULT FALSE;
DECLARE _TitleID int;
DECLARE _AuthorID int;
DECLARE _Royalty int;
DECLARE _AuthorCount int;
DECLARE _TotalSales decimal(7,2);
DECLARE _PaymentAmount decimal(7,2);
```

The cursor with the name `AllTitles` is declared. Notice that unlike the cursors you used in chapter 22, this cursor uses two columns instead of one:

```sql
DECLARE AllTitles CURSOR FOR

SELECT TitleID, AuthorID
FROM titleauthor
ORDER BY
    TitleID,
    AuthorOrder;
```

The handler variable `_Done` is declared to determine when to exit the cursor loop:

```sql
DECLARE CONTINUE HANDLER FOR NOT FOUND SET _Done = TRUE;
```

Then comes the `OPEN` the cursor, where the results of the query used by the cursor are retrieved:

```sql
OPEN AllTitles;
```

Next is the `LOOP` used by the cursor, named `GetTitles`:

```sql
GetTitles: LOOP
```

The first results of the cursor are fetched, and the values are assigned to the variables `_TitleID` and `_AuthorID`:

```sql
    FETCH AllTitles INTO _TitleID, _AuthorID;
```

After that, values are assigned to other variables. The first assignment is the value for `_Royalty`, which is assigned for the particular `_TitleID` value. You probably don't need the `_Royalty` variable. You could just as easily use the value in the title table instead of populating and using this variable:

```sql
    SET _Royalty = (
        SELECT Royalty
        FROM title
        WHERE TitleID = _TitleID
        );
```

The value for `_AuthorCount` is determined for the particular `_TitleID`. Having a separate query determine this value may not be a bad idea, although if you use a `GROUP` `BY` on the titleauthor table and group by TitleID, you might be able to use an `INNER` `JOIN` to include the Royalty amount noted earlier as well. Because Royalty is a column in the title table, you know that a one-to-one relationship exists with the results grouped by TitleID in the titleauthor table:

```sql
    SET _AuthorCount = (
        SELECT COUNT(AuthorID)
        FROM titleauthor
        WHERE TitleID = _TitleID
        );
```

The value for `_TotalSales`, which represents sales in terms of dollars, is determined by taking the `SUM` of the Quantity multiplied by the Price for the title for all orders placed in the `_PaymentYear`. The rows that match the `_PaymentYear` are calculated by using the `YEAR` function to determine the year of every value of the OrderDate column in the orderheader table. It makes sense to have this calculation as a separate query, but you probably should avoid using the `YEAR` function this way. Although they appear to be convenient, as noted in chapter 14, functions can be inefficient if you have millions of rows to evaluate in tables:

```sql
    SET _TotalSales = (
        SELECT SUM(orderitem.Quantity * orderitem.ItemPrice)
        FROM orderheader
        INNER JOIN orderitem
            ON orderheader.OrderID = orderitem.OrderID
        WHERE orderitem.TitleID = _TitleID
            AND YEAR(orderheader.OrderDate) = _PaymentYear
        );
```

In the last variable assignment, the calculation to determine the value for `_PaymentAmount`, which is the amount to be paid to the author based on their royalty, is calculated using the `_TotalSales`, `_Royalty`, and `_AuthorCount` values. You're getting to the heart of the cursor used by the stored procedure, and it seems that apart from the values for `_TotalSales` and `AuthorCount`, a lot of unnecessary queries are being used to determine values because the author wasn't thinking about a set-based solution:

```sql
    SET _PaymentAmount =
        COALESCE(CONVERT(
            ((_TotalSales * (_Royalty/100))/_AuthorCount), decimal(7,2))
            , 0.00);
```

If the value for `_PaymentAmount` is greater than 0, a row representing the payment is inserted into the authorpayment table. Although this approach is necessary with a cursor, if you used a set-based approach, you wouldn't need this `IF...THEN` statement. The results from properly used `INNER` `JOIN`s would exclude any titles and authors that did not have any titles sold, and therefore would result in no payment:

```sql
    IF _PaymentAmount > 0.00 THEN
        INSERT authorpayment (
            AuthorID,
            TitleID,
            PaymentYear,
            PaymentAmount
            )
        SELECT
            _AuthorID,
            _TitleID,
            _PaymentYear,
            _PaymentAmount;
        END IF;
```

If you've fetched all the values for the rows in your cursor, here is where you exit the loop:

```sql
        IF _Done = TRUE THEN

            LEAVE GetTitles;
        END IF;
```

At the end of the stored procedure, you end the `LOOP`, close the cursor, use `END` to represent the end of all actions in the stored procedure, and then change the delimiter back to the standard semicolon:

```sql
    END LOOP GetTitles;

    CLOSE AllTitles;

END //

DELIMITER ;
```

After reviewing the entire stored procedure, you should be able to make improvements that eliminate the use of a cursor, which makes the relational database management system (RDBMS) do less work and also eliminates the need for any variables.

### 23.2.3 Improving the INSERT stored procedure

The first thing you want to do is rewrite the sections containing queries you want to keep. Your review noted that you could use a query to determine the count of authors for each title (used to calculate the royalty payment) and include the Royalty values from the title table as well. The query, which will be used in a subquery, might look like this:

```sql
    SELECT
        t.TitleID,
        t.Royalty,
        COUNT(ta.AuthorID) AS AuthorCount
    FROM title t
    INNER JOIN titleauthor ta
        ON t.TitleID = ta.TitleID
    GROUP BY
        t.TitleID,
        t.Royalty
```

The review also revealed that you could use the logic to determine the sales of a title per year in dollars in a subquery. You want to avoid using a function on the OrderDate column of the orderheader table, which would cause extra work for the RDBMS. You can use some different date functions to take the value for year and then calculate starting and ending dates for the value of the `_PaymentYear` parameter.

The first date function is `MAKEDATE`, which allows you to make a date of the first day of the year using only a value for the year. You'll set the date for the first day of the chosen year like this:

```sql
MAKEDATE(_PaymentYear, 1)
```

You could use `MAKEDATE` to select the last day of the year by replacing the value 1 with 365, but that doesn't work for every year. Leap years have 366 days, and because you don't know if the value passed for `_PaymentYear` is a leap year, it would be better to calculate the end of your date range as anything less than the start of the next year. To calculate that, use the `DATE_ADD` function like this:

```sql
DATE_ADD(MAKEDATE(@Year, 1), INTERVAL 1 YEAR)
```

##### Note

Although not all RDBMSs have these specific functions, they all have similar functions with different names that can help you determine a date from parts such as the year, as well as functions that help you make calculations with dates.

With the range of dates that will replace the `YEAR` function sorted, here's the query to determine total sales per title, which will also be used in a subquery:

```sql
    SELECT
        oi.TitleID,
        SUM(oi.Quantity * oi.ItemPrice) AS TotalSales
    FROM orderheader oh
    INNER JOIN orderitem oi
        ON oh.OrderID = oi.OrderID
    WHERE oh.OrderDate >= MAKEDATE(@Year, 1)
        AND oh.OrderDate < DATE_ADD(MAKEDATE(@Year, 1), INTERVAL 1 YEAR)
    GROUP BY
        oi.TitleID
```

You need one more query that calculates the payment amount. Having the payment amount value for each AuthorID and TitleID in the chosen year allows you to populate the authorpayment table. Because both of the two preceding queries to be used for subqueries include TitleID, they should be easy to join.

Using the logic to calculate the payment amount from the original stored procedure, you can put all the logic together in a stored procedure with one query. Because a bit of complexity is involved, add a few comments to your stored procedure to explain your intentions:

```sql
DELIMITER //

CREATE PROCEDURE InsertAnnualPayment(
    IN _PaymentYear int
    )
BEGIN

INSERT authorpayment (
    AuthorID,
    TitleID,
    PaymentYear,
    PaymentAmount
    )
/* Calculate the total royalty per author */
SELECT
    ta.AuthorID,
    ta.TitleID,
    _PaymentYear,
    CONVERT((
        (sales.TotalSales * (royalty.Royalty/100))/royalty.AuthorCount),
        decimal(7,2)) AS RoyaltyPerAuthor
FROM titleauthor ta
INNER JOIN (
    /* Determine annual sales by title */
    SELECT
        oi.TitleID,
        SUM(oi.Quantity * oi.ItemPrice) AS TotalSales
    FROM orderheader oh
    INNER JOIN orderitem oi
        ON oh.OrderID = oi.OrderID
    WHERE oh.OrderDate >= MAKEDATE(@Year, 1)
        AND oh.OrderDate < DATE_ADD(MAKEDATE(@Year, 1), INTERVAL 1 YEAR)
    GROUP BY
        oi.TitleID
    ) sales
    ON ta.TitleID = sales.TitleID
INNER JOIN (
    /* Determine the royalty and count of authors */
    SELECT
        t.TitleID,
        t.Royalty,
        COUNT(ta2.AuthorID) AS AuthorCount
    FROM title t
    INNER JOIN titleauthor ta2
        ON t.TitleID = ta2.TitleID
    GROUP BY
        t.TitleID,
        t.Royalty
        ) royalty
    ON ta.TitleID = royalty.TitleID;

END //

DELIMITER ;
```

Now you have a stored procedure with no cursor, no variables, and no queries beyond what you need. You can populate the authorpayment table using set-based programming, which should be your goal whenever you write SQL. You don't have to worry about mismatched data types, and you aren't using functions in ways that could affect performance.

The stored procedure is greatly improved. But could you improve it even more?

### 23.2.4 Improving the INSERT stored procedure even more

You've learned a lot in 23 chapters, so don't be afraid to consider other keywords and techniques when you make further improvements in someone else's script. You may be able to improve a few more parts of this stored procedure, starting with the subqueries.

Although the two subqueries in this new version of the stored procedure perform better than the cursor, this solution might not be the best one if your orderheader and orderitem tables contain millions of records. It may be a good idea to replace the subquery that calculates the `TotalSales` for each title in a given year with a temporary table. Although this approach means writing more data to a temporary table, the subsequent `INNER` `JOIN` may read less data than all the data read in the subquery. Although using temporary tables isn't required for the current data set, which has only a few rows in each table, in many situations, a bit of testing will reveal that using temporary tables can help performance.

Another possible improvement is related to the `_PaymentYear` parameter. The single `_PaymentYear` parameter used by the stored procedure doesn't offer much flexibility, so you might consider replacing it with parameters for the start and end dates of a range of dates. This approach allows for annual, quarterly, monthly, and even custom ranges of values. Consider making any script more flexible if you can, anticipating that there'll be more diverse query requests in the future.

Finally, depending on the use of this data, it may be worthwhile to replace the InsertAnnualPayment stored procedure and the authorpayment table with a view that calculates the author and title royalty for every sale. Remember that a view is simply a stored query to be called on whenever you want. If you remove the consideration for a specific date range, you could build a query to calculate the royalty of every orderitem with a query like this:

```sql
SELECT
    ta.AuthorID,
    ta.TitleID,
    oh.OrderID,
    oh.OrderDate,
    CONVERT((
        (SUM(oi.Quantity * oi.ItemPrice) * (t.Royalty/100))/ac.AuthorCount),
        decimal(7,2)) AS RoyaltyPerAuthor
FROM title t
INNER JOIN titleauthor ta
    ON t.TitleID = ta.TitleID
INNER JOIN orderitem oi
    ON ta.TitleID = oi.TitleID
INNER JOIN orderheader oh
    ON oi.OrderID = oh.OrderID
INNER JOIN (
    /* Determine the royalty and count of authors */
    SELECT
        TitleID,
        COUNT(AuthorID) AS AuthorCount
    FROM titleauthor
    GROUP BY
        TitleID
        ) ac
    ON ta.TitleID = ac.TitleID
 GROUP BY
    ta.AuthorID,
    ta.TitleID,
    oh.OrderID
    oh.OrderDate;
```

This view would eliminate the redundancy of all the data in the authorpayment table. It would also allow you to query the view for whatever values of AuthorID, TitleID, and OrderID you want, as well as query any range of OrderDate values. A view doesn't allow you to persist the data the way the authorpayment table does, but if you don't need the data to be persisted, a view is a great option.

The key idea that you should take away from this chapter is this: you have options. Throughout this month of lunches, you've learned dozens of keywords and concepts, and you've seen how to use them effectively. I hope that reviewing these scripts was helpful and that it gave you a higher level of confidence in your ability to use the SQL language.

