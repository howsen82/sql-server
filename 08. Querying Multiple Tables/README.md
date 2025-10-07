# 8 Querying Multiple Tables

Back in chapter 2, we discussed how relational database management systems (RDBMSes) store data in objects known as *tables*, and since then, we've been examining ways to query these tables. I don't know whether you've been wondering what makes an RDBMS "relational," but I'm going to answer that very question.

One of the primary features of an RDBMS is that it allows a set of data to be stored so that it can relate to other sets of data—hence, the use of the word *relational*. This way of storing data is incredibly powerful because we can not only put the data we have in logical groupings of tables but can also easily retrieve related data from multiple tables with a single query.

Retrieving data this way is done by *joining tables*, which means combining the data in two tables using the values that form the relationship between the tables. Although joining tables is common and relatively easy, you must follow some specific rules to get the desired results. You'll soon learn those rules and see how to write join tables in SQL correctly. First, though, you need to consider a few vital concepts of relational databases and the way they are designed.

## 8.1 The rules of data relationships

We haven't focused on the words *relational* or *relationship,* but we looked at one aspect of relationships in an RDBMS when we started looking at rows in tables in chapter 2. Think about a single row from any table: it's a collection of values that all relate to one another. The first row in the title table, for example, contains values such as TitleID, TitleName, and others that relate to the title "Pride and Predicates," so those values all relate to one another. It's the same for every row in the table, except that each row represents related values for a different title.

Although we haven't looked at any examples, values can also relate to other rows in other tables. We've taken all the information specifically related to a single title into the title table, but in the sqlnovel database, we have information that relates to titles elsewhere. One of the tables used to track information about orders of different titles is orderitem. Try the following query, and take a look at the results in figure 8.1:

```sql
SELECT *
FROM orderitem;
```

##### Figure 8.1 The first 10 rows in the orderitem table, including the column TitleID

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH08_F01_Iannucci.png)

The third column is named TitleID, which is the same as the first column in the title table. This name indicates that values in the orderitem table relate to the values in the title table, which makes sense because the titles are what customers are ordering.

But why would we store these values in different tables instead of putting the title-related values we need in the orderitem table? Well, there are several good reasons for storing the data in separate tables. But rather than simply describe these reasons, it might be more helpful to give you an example that shows the reasons using some of the data you've already worked with.

### 8.1.1 Data without relationships

Suppose that we design a version of the sqlnovel database to track orders, and all the necessary data is stored in this single hypothetical orders table. This table will have columns for order date, title name, price, and customer's first and last names. The table might look something like figure 8.2.

##### Figure 8.2 Our hypothetical table, used to track orders, that contains order date, title name, price, and customer name

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH08_F02_Iannucci.png)

On the surface, this table appears to be a logical way to track orders, and perhaps you've used a spreadsheet similarly. The table may be fine for tracking a small number of orders, but a closer look at the data reveals quite a few redundant values. In this table, we see what appear to be orders for the same two titles placed by two different customers on different dates. The main problem isn't so much that we're using five rows of data to represent these orders, but that we have to repeat the data values so often.

Imagine that this table has millions of rows. You can see that over time, it could become a problematic waste of query time and storage. This is especially true for the string values of TitleName and customer FirstName and LastName because string values generally take much more storage space than numeric values do.

That problem isn't the only one, though. What would happen if any values of data changed, such as a customer's last name? If a customer changed their last name and placed a new order under their new name, how would we connect the orders placed under the previous name to those placed under the new name? We couldn't do that with this table; with different last names, the orders would appear to be placed by different customers.

The same problem could exist if the data was entered incorrectly. How could we track sales if the last TitleName was inadvertently entered as "The Join Luck Clubs" (with an extra s added to Club)? We couldn't; that value would be a different one. Even though you and I can see that this title is a data-entry mistake, in the data, that entry would be a different title, and it wouldn't show up in results if we wrote a SQL query that used `WHERE TitleName` `=` `'The` `Join` `Luck` `Club'` as a filter.

As you can see, storing all this data in a single table can lead to lots of problems. Let's look at ways to use some basic relational database concepts to organize this data better.

### 8.1.2 Data with relationships

In a relational database, we want to eliminate as many redundant occurrences of the same values as possible. We can accomplish this by doing a few things:

* *Organize the data in logical groups of values.* We put these values in separate tables, and we want each row in each table to relate to something unique, such as the title of a book. Think again about how any given row in the title table contains data this way.
* *Determine what column or columns will contain a unique value in each of our new tables.* This column or set of columns will be known as the *primary key*, which allows us to relate data in other tables to this table. In the title table, the primary key is the TitleID column.
* *Replace the data in other tables with these primary-key values to represent the values we want to reference.* Because these key values in our orders table relate to values in other tables, they will be known as *foreign keys*. In the table in figure 8.2, we'll start by replacing the TitleName column with the corresponding TitleID values from the title table because TitleID is the key value.

Let's do this with our orders table. Start by looking again at figure 8.2 to see how to organize the data this way.

First, we have repeating values for TitleName, so it makes sense to create a separate table that stores these title values. The good news, as you've no doubt noticed, is that we already have a title table in our sqlnovel database that stores the data this way. Let's look at the title-table values for TitleID and TitleName for the two titles in our orders table, shown in figure 8.3:

```sql
SELECT
    TitleID,
    TitleName
FROM title
WHERE TitleName IN ('Pride and Predicates', 'The Join Luck Club')
ORDER BY TitleID;
```

##### Figure 8.3 The results for TitleID and TitleName for the titles Pride and Predicates and The Join Luck Club

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH08_F03_Iannucci.png)

##### Note

The values for TitleID in the title table must be unique so that we know exactly which row in the title table to reference. If the TitleID values are duplicated, the data becomes inconsistent and confusing; we don't know which values are being referenced.

The TitleID column serves as the primary key, so we can replace the TitleName column in our orders table with TitleID, which corresponds to values in the title table. Figure 8.4 shows what our hypothetical table looks like now.

##### Figure 8.4 What it would look like if we replaced TitleName with TitleID in our hypothetical orders table

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH08_F04_Iannucci.png)

Now we have a relationship between these tables, which allows us to avoid the problems we discussed earlier—at least as they relate to titles and their names. We're saving space by storing a smaller numeric value instead of a string each time we want to refer to the title. We also have less of a problem with data inconsistency because a single source stores the title name. If any other tables want to reference a particular title name, they too can use the TitleID values.

##### Note

This is what makes RDBMSes popular for storing many kinds of data. Storing values in a relational way allows us to store data efficiently, change values easily, and (most important) keep the data consistent throughout the database.

If we look at our orders table, we can consider other ways to store data more efficiently. Customers are unique individuals, so any customer data should be in a separate table, with the data in our orders table being replaced by a key value. Again, we already have a customer table structured with these values and a primary key of CustomerID. Figure 8.5 shows the results of our query of the customer table for the FirstName and LastName values in the orders table:

```sql
SELECT
    CustomerID,
    FirstName,
    LastName
FROM customer
WHERE (FirstName = 'Chris' AND LastName = 'Dixon')
    OR (FirstName = 'David' AND LastName = 'Power')
ORDER BY CustomerID;
```

##### Figure 8.5 The results from the CustomerID, FirstName, and LastName columns for customers with the name Chris Dixon or David Power

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH08_F05_Iannucci.png)

Now we have three related tables, so let's replace the two customer-name columns in our orders table with a single column referencing the corresponding CustomerID values from the customer table. Figure 8.6 shows the updated orders table.

We have a *one-to-many* relationship between the customer table and our orders table. That means that for each order, we have one customer, but any given customer can have more than one order. This type of relationship is common in relational databases.

Our data is organized even more efficiently, but we can make one last change. Consider the third and fourth rows in figure 8.6. It looks like Customer 1 purchased two different items on the same day, which for the purposes of our exercise are considered the same order. This makes sense, and we should expect that many times, customers will order more than one item in a given order.

##### Figure 8.6 Our hypothetical orders table with a CustomerID column to reference names in the customer table

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH08_F06_Iannucci.png)

This poses a problem for creating a unique primary key for our orders table, however, because we can't place a unique key for orders on the rows if there are duplicate rows for any given order. One common way to resolve this problem is to place the items ordered in a separate table, thus dividing the data related to an order into two tables. Because any order can include one or more items, this relationship is also considered a one-to-many relationship.

##### NOTE

Relational databases also have *one-to-one* and *many-to-many* relationships between tables. Generally, these relationships are less common, so we won't look at any examples now. Just know that other kinds of relationships can exist between tables in any database.

If we're going to divide the data in our orders table, we need to consider the following question for all the columns: Do the values relate to a specific item in the order or generally to the order itself? Let's examine the columns:

* *OrderDate*—These values are the same for the entire order because all items are ordered at the same time in a given order.
* *TitleID*—The values are specific to an item because an order can contain more than one title.
* *Price*—This value relates to individual titles, so it is also an item-level value.
* *CustomerID*—This value relates to the entire order because the customer is the same for all items in an order.

Now that we've identified what values go into which tables, we can divide the values in our orders table into two separate tables:

* *orderheader*—Contains the values unique to the entire order. We'll create a primary-key column called OrderID in the orderheader table. This table will also include columns for OrderDate and CustomerID.
* *orderitem*—Contains the values unique to the items in a given order. We'll create a primary-key column called OrderItemID in the orderitem table and a foreign-key column called OrderID to create a relationship between orderitem and orderheader. This table will also include columns for TitleID and Price.

We have organized the data in our hypothetical orders table into the actual tables in the sqlnovel database, and we understand how these tables relate to one another. Let's start writing SQL statements that join the data in these tables using their relationships.

## 8.2 The way to join data

Now that you have a basic understanding of how tables and keys are used, let's see how they're used in queries. To do this, we'll focus on the `FROM` clause in queries, which is where we identify the data set to be used.

### 8.2.1 Joining two tables

If we want to find out which customer placed the first order, which is OrderID 1001, we might start with a query like this:

```sql
SELECT CustomerID
FROM orderheader
WHERE OrderID = 1001;
```

##### Try it now

You've waited long enough to write some SQL in this chapter, so execute that query.

There's probably not a lot of value in showing a picture of a single column with a single value in the result set, so if you want to keep reading instead of executing the query, know that the value returned for CustomerID is 1.

Knowing that the CustomerID for the first order is 1 may be useful for a lot of queries, but suppose that we want to know that customer's name. For this task, we need to use the relationship between the orderheader and customer tables by *joining* them in our query. We do this by explicitly stating the second table name (customer) and the column names common to both tables (CustomerID), which we'll use to join the data. We state all this in the `FROM` clause, using the keywords `JOIN` and `ON`.

The following query uses `JOIN` and `ON` to join the orderheader and customer tables to return not only the CustomerID but also the customer's first and last names, as shown in figure 8.7:

```sql
SELECT
    orderheader.CustomerID,
    customer.FirstName,
    customer.LastName
FROM orderheader
JOIN customer
    ON orderheader.CustomerID = customer.CustomerID
WHERE orderheader.OrderID = 1001;
```

##### Figure 8.7 The results of joining the orderheader and customer tables to show CustomerID and customer-name values for the first order

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH08_F07_Iannucci.png)

Let's take a closer look at this query. The first thing you may notice is that the `JOIN` comes after `FROM`, and `ON` comes after that. The `JOIN` keyword is considered part of the `FROM` clause; it tells our RDBMS that we want to use more data in another table.

Think of using `FROM` and `JOIN` a bit like using `WHERE` and `AND` for filtering. If we have multiple filtering conditions for the `WHERE` clause, we start with the keyword `WHERE` to state the first condition, but every subsequent condition starts with `AND`. Similarly, in the `FROM` clause, we start with `FROM` and then declare the first table from which we want data, which in this query is orderheader. Then, because we also want data from a second table, we connect the data between the two tables by using the `JOIN` keyword followed by the name of the second table, which in this query is customer. Any subsequent filtering conditions in the `WHERE` clause will use the `AND` keyword, and any subsequent table joins in the `FROM` clause will use the `JOIN` keyword.

Merely stating that we want to `JOIN` the tables isn't enough, though, so we also need to say *how* we're relating these tables by explicitly stating the columns we're using to establish the relationship. We do this by using the `ON` keyword in a predicate. A *predicate* is any part of our SQL statement that evaluates whether something is true, false, or unknown, which is what the `ON` section of the join does: finds all rows that match CustomerID in each table. If the match is true, the rows are considered for inclusion in our result set.

Although I haven't mentioned it yet, the filtering condition in the `WHERE` clause of our query is also considered a predicate because we're also asking the RDBMS to evaluate `WHERE` `orderheader.OrderID` `=` `1001`. Like that condition, our `JOIN` is considered an *exclusive condition*, so after evaluating the predicates in our join and our filtering conditions, only those that are considered true matches are returned. We have only one row in our results because only that row's values meet all our conditions.

Also notice that in the `ON` part of our query, we're using *two-part names* for our columns. This name refers to the syntax of `[table name].[column name]`, and it's crucial because the CustomerID column is included in both tables. We can't simply say `ON` `CustomerID =` `CustomerID` because the RDBMS won't know which CustomerID column we mean. If you think this fact is obvious in the example, rest assured that many databases have tables with columns that relate to one another but that have different column names. For this reason, we need to use two-part names including both the table and column.

##### TIP

Although it doesn't matter which table and column is mentioned first in the `ON` portion of this `JOIN`, it's a good idea to start with the table mentioned first. This approach helps with readability and data organization, of course, but as you'll see in chapter 9, it's also crucial for working with other kinds of joins.

These two-part names for columns end up being used throughout the query, mostly for readability. I say *mostly* because we could change almost all the two-part names to use only the column name and not the table name except for any time we use CustomerID. This is because the CustomerID column exists in both tables, so if our query said `CustomerID` anywhere without a reference to the table, we'd get an error saying that the `CustomerID` reference is ambiguous.

##### Try it now

Execute the query used to get the result in figure 8.7. Also change `orderheader.CustomerID` in the `SELECT` clause to just `CustomerID`, and see the error that results in the Output panel.

### 8.2.2 Joining more tables

The great thing about joining tables is we aren't limited to two tables. We can continue to use `JOIN` to connect more data provided that we know the correct columns used for the relationships between tables.

Recall that we organized our order-specific data into two tables: orderheader and orderitem. If we want to find even more information about the first order, such as the price of the item purchased, we could easily add another join to our query. Because we established previously that the orderheader and orderitem tables will be joined on OrderID, which is the primary key for the orderheader table, we can modify our query with a few more lines of SQL related to the orderitem table. Figure 8.8 shows the results of this query:

```sql
SELECT
    orderheader.CustomerID,
    customer.FirstName,
    customer.LastName,
    orderitem.ItemPrice
FROM orderheader
JOIN customer
    ON orderheader.CustomerID = customer.CustomerID
JOIN orderitem
    ON orderheader.OrderID = orderitem.OrderID
WHERE orderheader.OrderID = 1001;
```

##### Figure 8.8 Customer information from the customer table and the price of the item ordered in the first order from the orderitem table

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH08_F08_Iannucci.png)

In this query, the `JOIN` for the orderitem table comes after the `JOIN` for customer, but in this particular query, the order of these joined tables doesn't matter. We could just as easily have written the query with the `JOIN` for orderitem occurring before the `JOIN` for customer. The arrangement of the order of tables joined comes down to personal preference and readability, so long as both tables used in any `JOIN` have been declared in the `FROM` clause by the time we get to the `ON` portion of the join.

We can add one more table to get the name of the item that was ordered because we have the value for TitleName in a fourth table: title. If you recall, the TitleName is referenced in the orderitem table by the TitleID foreign key, which means that we must include our `JOIN` to the title table after orderitem. Here's the query we'll use to get the results shown in figure 8.9:

```sql
SELECT
    orderheader.CustomerID,
    customer.FirstName,
    customer.LastName,
    orderitem.ItemPrice,
    title.TitleName
FROM orderheader
JOIN customer
    ON orderheader.CustomerID = customer.CustomerID
JOIN orderitem
    ON orderheader.OrderID = orderitem.OrderID
JOIN title
    ON orderitem.TitleID = title.TitleID
WHERE orderheader.OrderID = 1001;
```

##### Figure 8.9 Customer information from the customer table, the price of the item ordered in the first order from the orderitem table, and the TitleName from the title table

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH08_F09_Iannucci.png)

We can keep joining more tables to get related data, and in later chapters, as we discuss more of the tables in our database, we'll do just that. But as you can see, our queries with all these two-part names make for a lot of words in our SQL query. Even for seasoned query writers, this approach is a wordy way to write a query with a few joins. A much more readable way to write these two-part names involves a familiar concept.

## 8.3 Table aliases

Recall that in chapter 3, we talked about renaming columns in our result set by using *aliases*. We effectively declared that a column would be referenced, at least in the output, but with a different name. Fortunately, the SQL language allows us to use aliases for table names as well.

Our goal in using table aliases is different from our usual goal with column aliases. With columns, we often want to change the column name in the results to be more descriptive, but with table aliases, we want to be less descriptive. Generally, we use an alias of one or two characters to reduce the overall number of characters in our query—which, if done correctly, makes our query easier to read.

One common way to alias the table names is to use one- or two-letter abbreviations for the tables being aliased. We can use an alias of `c` for the customer table or `t` for the title table, for example. We could use `o` as an alias for the orderheader table, but because another table in our query starts with `o` (orderitem), we can use two-letter aliases for those tables instead. One logical way to use an alias for these tables is to use the first letter of each word in the table names, such as `oh` for orderheader and `oi` for orderitem. Here's an example of these types of aliases using the preceding query:

```sql
SELECT
    oh.CustomerID,
    c.FirstName,
    c.LastName,
    oi.ItemPrice,
    t.TitleName
FROM orderheader oh
JOIN customer c
    ON oh.CustomerID = c.CustomerID
JOIN orderitem oi
    ON oh.OrderID = oi.OrderID
JOIN title t
    ON oi.TitleID = t.TitleID
WHERE oh.OrderID = 1001;
```

Fewer characters fill the screen in that query, which means that we have less information to review if we want to understand this query. We could alias these table names the same way we aliased columns if we wanted to, of course. We could alias orderheader, for example, by saying `FROM` `orderheader` `AS` `oh`. But this type of aliasing isn't done often because our goal in aliasing table names is to reduce the overall number of characters in a query. For this reason, I highly encourage you to use table aliases as shown here when you join tables in any query because aliases make your queries much easier to read. We have to follow a couple of rules for table names, however:

* Aliases must start with an alphabetical character, not a number or a special character.
* Except for the first character, an alias can contain numbers but not special characters.

The only other consideration for aliases is making them sensible. Don't alias the first table as `a`, the second table as `b`, and so on. If your aliases at least remotely represent the actual table names, your SQL queries will be much easier to read and understand.

##### Try it now

Rewrite any of the queries in section 8.2 to use your own aliases.

## 8.4 The other way to join data

Earlier in the chapter, I discussed predicates and showed how they evaluate conditions in the joins we use in our `FROM` clause and the conditions we state in the `WHERE` clause. Although this method is rarely used, there's a way to combine all the predicates in the `WHERE` clause.

I mention this technique only because at some point, you'll likely encounter a SQL query written by someone else who uses it for joins. As you'll soon see, this technique is generally discouraged for several reasons. Here's what the preceding query would look like if we used this other way of joining:

```sql
SELECT
    oh.CustomerID,
    c.FirstName,
    c.LastName,
    oi.ItemPrice,
    t.TitleName
FROM
    orderheader oh,
    customer c,
    orderitem oi,
    title t
WHERE oh.OrderID = 1001
    AND oh.CustomerID = c.CustomerID
    AND oh.OrderID = oi.OrderID
    AND oi.TitleID = t.TitleID;
```

The first thing you might notice is that with this method of joining data, we have an easily readable, comma-separated list of tables in the `FROM` clause. This format is a bit closer to the verbal English we've been considering throughout the book because we might verbally declare the intentions for this query something like this: "I would like the customer ID, first name, last name, item price, and title name from the orderheader table, customer table, orderitem table, and title table where the order ID is 1001."

But even this method is difficult to convert from a verbal declaration to SQL because we have to tell the RDBMS exactly how all those tables need to be joined. Although this method is a perfectly valid way to join data in SQL, it has some disadvantages:

* When it comes to finding how all these tables are joined, we have to read every row in the `WHERE` clause to determine how any single join is evaluated. This query may use fewer characters because it doesn't say `JOIN` for each join, but for this query and more complex queries, we have to scan the entire `WHERE` clause to find out how any two tables are joined. Combining all these evaluations in the `WHERE` clause makes troubleshooting much more difficult—like trying to find a particular noodle in a bowl of spaghetti.
* This method allows only a particular type of join. In chapter 9, we'll discuss more-inclusive ways to use `JOIN` to connect data. We won't be able to connect data in these inclusive ways using this method. For these reasons, you should avoid writing SQL that contains joins in the `WHERE` clause.

Joins will be critical components of nearly every query you'll write from now on, so if you're unsure how they work, please take time to review this chapter and practice the query examples starting in section 8.2, as well as the following lab exercises. When you're feeling confident about joining tables, I'll see you in chapter 9!

## 8.5 Lab

1.  What is the difference in the output of the following two queries, which use different tables for filtering in the `WHERE` clause?

```sql
SELECT
    t.TitleName
FROM orderheader oh
JOIN customer c
    ON oh.CustomerID = c.CustomerID
JOIN orderitem oi
    ON oh.OrderID = oi.OrderID
JOIN title t
    ON oi.TitleID = t.TitleID
WHERE oh.OrderID = 1001;

SELECT
    t.TitleName
FROM orderheader oh
JOIN customer c
    ON oh.CustomerID = c.CustomerID
JOIN orderitem oi
    ON oh.OrderID = oi.OrderID
JOIN title t
    ON oi.TitleID = t.TitleID
WHERE oi.OrderID = 1001;
```

2.  How many orders did the customer named Chris Dixon place? Write a query to determine the answer.

3.  What are the names of the customers who ordered a title in 2015? Write a query to determine the answer.

4.  How could you rewrite the following query, which finds the names of all customers who ordered "The Sum Also Rises," using `JOIN`s and aliases?

```sql
SELECT
    customer.FirstName,
    customer.LastName
FROM title, orderheader, customer, orderitem
WHERE title.TitleName = 'The Sum Also Rises'
    AND orderheader.OrderID = orderitem.OrderID
    AND orderitem.TitleID = title.TitleID
    AND orderheader.CustomerID = customer.CustomerID;
```

5.  We saw in section 8.4 that we can move all the predicates to the `WHERE` clause. What happens if we move all the predicates to the `FROM` clause, as in the following query?

```sql
SELECT
    t.TitleName
FROM orderheader oh
JOIN customer c
    ON oh.CustomerID = c.CustomerID
JOIN orderitem oi
    ON oh.OrderID = oi.OrderID
    AND oh.OrderID = 1001
JOIN title t
    ON oi.TitleID = t.TitleID;
```

## 8.6 Lab answers

1.  The results are the same. Because our query is matching all OrderID values from the orderheader to the orderitem table, the OrderID column from either table can be used in the filtering condition to return the same results.

2.  Chris Dixon placed three orders. You could use a query like this one to get the results:

```sql
SELECT
    oh.OrderID
FROM orderheader oh
JOIN customer c ON oh.CustomerID = c.CustomerID
WHERE c.FirstName = 'Chris'
    AND c.LastName = 'Dixon';
```

3.  Eight customers placed an order in 2015:

•   Chris Dixon

•   David Power

•   Arnold Hinchcliffe

•   Keanu O'Ward

•   Lisa Rosenqvist

•   Maggie Ilott

•   Cora Daly

•   Dan Wilson

You could find them with a query like this:

```sql
SELECT
    c.FirstName,
    c.LastName,
    oh.OrderDate
FROM orderheader oh
JOIN customer c ON oh.CustomerID = c.CustomerID
WHERE oh.OrderDate >= '2015-01-01 00:00:00'
AND oh.OrderDate < '2016-01-01 00:00:00';
```

4.  You could write a query like this using `JOIN`s and aliases:

```sql
SELECT
    c.FirstName,
    c.LastName
FROM orderheader oh
JOIN customer c
    ON oh.CustomerID = c.CustomerID
JOIN orderitem oi
    ON oh.OrderID = oi.OrderID
JOIN title t
    ON oi.TitleID = t.TitleID
WHERE t.TitleName = 'The Sum Also Rises';
```

5.  The query executes successfully with the same result set, but as noted in this chapter, we generally try to avoid combining filtering predicates with join predicates for the sake of readability.

