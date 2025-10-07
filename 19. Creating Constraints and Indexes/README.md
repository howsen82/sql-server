# 19 Creating Constraints and Indexes

We talked about two important constraints in chapter 18: `PRIMARY` `KEY` and `FOREIGN KEY` constraints. This chapter looks at a few more constraints that help us ensure the integrity of the data in our tables.

We'll also discuss *indexes*, which are table-related objects that help with the performance of our queries. Just as indexes in books like this one can help you quickly find the subject you're looking for, indexes in a database can reduce the time it takes queries to find specific data.

I hope that you've enjoyed creating tables and associated constraints because you're about to create more.

## 19.1 Constraints

By completing the examples in chapter 18, you learned that you can create constraints in two ways: by creating them for an existing table using `ALTER` `TABLE` or by creating a new table using `CREATE` `TABLE`.

You have two ways to create a constraint using `CREATE` `TABLE`. First, you can create the constraint after all columns are declared, as you did in chapter 18. Here's an example of the `PRIMARY` `KEY` constraint you created for the category table:

```sql
CREATE TABLE category (
    CategoryID int,
    CategoryName varchar(20),
    CONSTRAINT PK_category PRIMARY KEY (CategoryID)
    );
```

You can also create a constraint as part of the declaration of the column after the data type has been declared. Here's an example of how you could have done that for the primary key of the category table:

```sql
CREATE TABLE category (
    CategoryID int PRIMARY KEY,
    CategoryName varchar(20)
    );
```

Although this approach is simpler, creating a constraint this way doesn't allow us to name the constraint. If we create the constraint this way, the name of the constraint is generated automatically by our relational database management system (RDBMS). This name will likely be some series of letters and numbers that doesn't indicate our intentions for the constraint.

That said, you don't necessarily need a name for every constraint. Although I highly recommend using the first method to create `PRIMARY` `KEY` and `FOREIGN` `KEY` constraints, other constraints in this chapter are typically created with the second method.

### 19.1.1 NOT NULL constraints

The `NOT NULL` constraint enforces a requirement that there cannot be null values in a column—which, if we think about it, is probably the case for most columns in any table. Tables are made to contain data, and we'll require many, if not all, columns in a table to have values.

Suppose that we want a table in our sqlnovel database that tracks shipments of novels to customers. We'll name the table shipment and add the following six columns:

* *ShipmentID*—Identifies each unique row in the table
* *OrderID*—Identifies the order to which the shipment is related
* *ShipmentCost*—Identifies the cost of the shipment in U.S. dollars
* *ShipmentMethod*—Identifies whether the order was sent via parcel post (P) or express (E)
* *TrackingNumber*—Identifies the tracking number provided by the shipment carrier
* *ShipmentDate*—Identifies the date when the shipment was sent

The data types for our columns will be

* *ShipmentID*—`int`, a unique integer value
* *OrderID*—`int`, the same as in the orderheader table
* *ShipmentCost*—`decimal(5,2)` to accommodate numbers from 0.00 to 999.99
* *ShipmentMethod*—`char(1)` because the value will be either P or E
* *TrackingNumber*—`varchar(20)` for whatever value the shipment carrier provides
* *ShipmentDate*—`datetime`, a data value

We also want to create a `PRIMARY` `KEY` constraint named PK_shipment on the ShipmentID column, and a `FOREIGN` `KEY` constraint named FK_shipment_orderheader on the OrderID column that references the OrderID values in orderheader. With all this information, we can create the shipment table using the following SQL statement:

```sql
CREATE TABLE shipment (
    ShipmentID int,
    OrderID int,
    ShipmentCost decimal(5,2),
    ShipmentMethod char(1),
    TrackingNumber varchar(20),
    ShipmentDate datetime,
    CONSTRAINT PK_shipment PRIMARY KEY (ShipmentID),
    CONSTRAINT FK_shipment_orderheader FOREIGN KEY (OrderID)
        REFERENCES orderheader(OrderID)
    );
```

##### Note

Don't execute this SQL yet. You'll modify it quite a bit in this chapter.

The next thing we want to determine is which of these columns will be *nullable*, which means that these columns can contain null values. For every column we determine to be not nullable, we want to add a `NOT` `NULL` constraint to ensure that all rows contain values for those columns. An example of a nullable column is the MiddleName column in the author table; some authors have a middle name and others don't. Consider each column in the shipment table:

* ShipmentID is not nullable because we can't have null values in a primary key.
* OrderID is not nullable because every shipment must relate to an order.
* ShipmentCost is not nullable because every shipment has a cost of $0.00 or more.
* ShipmentMethod is not nullable because we need to know how every shipment was sent.
* TrackingNumber is not nullable because each shipment has a tracking number.
* ShipmentDate is not nullable because we need to know when a shipment was sent.

After careful review, it looks as though none of these columns is nullable, so we should add a `NOT` `NULL` constraint to each column. We can do that by modifying our SQL to indicate which columns are `NOT` `NULL` after their data types are declared:

```sql
CREATE TABLE shipment (
    ShipmentID int NOT NULL,
    OrderID int NOT NULL,
    ShipmentCost decimal(5,2) NOT NULL,
    ShipmentMethod char(1) NOT NULL,
    TrackingNumber varchar(20) NOT NULL,
    ShipmentDate datetime NOT NULL,
    CONSTRAINT PK_shipment PRIMARY KEY (ShipmentID),
    CONSTRAINT FK_shipment_orderheader FOREIGN KEY (OrderID)
        REFERENCES orderheader(OrderID)
    );
```

By doing this, we ensure that every row will have a value for every column, which is what we want. If someone tries to enter a row that doesn't have a value for every column, they'll get an error message. This message will vary from one RDBMS to another, but in MySQL, it says that one of the columns doesn't have a default value.

What does this mean? Well, *default values* involve another kind of constraint.

### 19.1.2 DEFAULT constraints

`DEFAULT` constraints allow us to use a set default value for a column if no value is specified. Default values established by these constraints will be used whenever an `INSERT` statement doesn't indicate a value for columns with a `DEFAULT` constraint.

The `DEFAULT` constraint must be a *literal constant*, meaning that it will be the same expression for each row that's inserted. An expression is some combination of values, operators, or functions that are evaluated to another value. This expression could be a number, a date, or a string of characters, although in MySQL and most other RDBMSes, we can also use some date and time functions as the default.

Our shipment table can benefit from this kind of constraint with one of these functions. In chapter 14, we learned about the `CURRENT_DATE()` function, which returns the date and time for the immediate moment. We can use this function to make sure that whenever a row is inserted into our shipment table, it records the value for `CURRENT_DATE()` in the ShipmentDate column at the time the row is created.

##### Warning

Although it's fairly common, the `CURRENT_DATE()` function isn't available in every RDBMS. For SQL Server, use `GETDATE()`; for Oracle, use `SYSDATE`; and for SQLite, use `date('now')`.

For a `DEFAULT` constraint, we'll declare the keyword `DEFAULT`; then we'll declare the default value after declaring the data types for our column. Here's what our `CREATE TABLE` statement with this new default for ShipmentDate looks like:

```sql
CREATE TABLE shipment (
    ShipmentID int NOT NULL,
    OrderID int NOT NULL,
    ShipmentCost decimal(5,2) NOT NULL,
    ShipmentMethod char(1) NOT NULL,
    TrackingNumber varchar(20) NOT NULL,
    ShipmentDate datetime NOT NULL DEFAULT (CURRENT_DATE()),
    CONSTRAINT PK_shipment PRIMARY KEY (ShipmentID),
    CONSTRAINT FK_shipment_orderheader FOREIGN KEY (OrderID)
        REFERENCES orderheader(OrderID)
    );
```

Notice two points about this new constraint:

* *You need to put parentheses around the default value of* `CURRENT_DATE()` *in your* `CREATE` `TABLE` *statement.* The use of parentheses isn't required for most RDBMSes, but it is in MySQL. Consult the documentation for any RDBMS you're using to make sure that the syntax of your SQL statement is correct.
* *You can create more than one constraint on a column, as you're doing with the ShipmentDate column.* You don't even need to use a comma separator between columns, and you don't want to because the comma separator would indicate a new column, not a second constraint on the ShipmentDate column. Now this column has both a `NOT` `NULL` and a `DEFAULT` constraint, which is not uncommon for columns that automatically indicate the time when a row was added.

##### Note

Having a `DEFAULT` constraint on a column doesn't mean that we don't also need the `NOT` `NULL` constraint. The `DEFAULT` constraint guarantees that a value will be inserted if one is not specified, but we also need the `NOT` `NULL` constraint to ensure that we don't have NULL specified as a value at the time when the row is inserted.

### 19.1.3 UNIQUE constraints

Now let's look at another kind of constraint for a different column: the `UNIQUE` constraint. `UNIQUE` constraints enforce the requirement that any value in a column be unique in that column. If we insert or update a value for a column with a `UNIQUE` constraint and that value already exists in another row, an error will occur.

`UNIQUE` constraints are a bit like the `PRIMARY` `KEY` constraints we've already used, but there are a few exceptions. The main difference is that a table can contain only one `PRIMARY` `KEY` constraint, but it can contain multiple `UNIQUE` constraints on columns if necessary.

Unlike `PRIMARY` `KEY` constraints, `UNIQUE` constraints can include null values. The maximum number of null values in a column with a `UNIQUE` constraint depends on the RDBMS you're using; MySQL allows multiple null values, for example, whereas SQL Server and Oracle allow only one null value. If this restriction is a concern for you, consult the documentation for your RDBMS, although it's rare to have a column that requires a `UNIQUE` constraint and is still nullable.

Because tracking numbers are `UNIQUE` for the shipment carrier, we want to create a `UNIQUE` constraint on the TrackingNumber column to ensure that a duplicate value is never used for this column. Adding this constraint is as simple as adding the word `UNIQUE` to the column declaration, similar to what we did with other constraints in this chapter:

```sql
CREATE TABLE shipment (
    ShipmentID int NOT NULL,
    OrderID int NOT NULL,
    ShipmentCost decimal(5,2) NOT NULL,
    ShipmentMethod char(1) NOT NULL,
    TrackingNumber varchar(20) NOT NULL UNIQUE,
    ShipmentDate datetime NOT NULL DEFAULT (CURRENT_DATE()),
    CONSTRAINT PK_shipment PRIMARY KEY (ShipmentID),
    CONSTRAINT FK_shipment_orderheader FOREIGN KEY (OrderID)
        REFERENCES orderheader(OrderID)
    );
```

### 19.1.4 CHECK constraints

We want to use one final constraint in our table: the `CHECK` constraint. A `CHECK` constraint allows us to limit the values used in a column by comparing them to some kind of expression. Being able to use an expression in the `CHECK` constraint gives us quite a bit of flexibility in evaluating the validity of values for any given column.

For the shipment table, we want to add `CHECK` constraints to the ShipmentCost and ShipmentMethod columns. The ShipmentMethod column requires a value that's either E or P, so we'll write the expression to be used in our constraint as `ShipmentMethod` `IN` `('P',` `'E')`.

##### Warning

The expressions we use in `CHECK` constraints can include multiple columns, so we must state the column (or columns) used in our expressions.

For the `CHECK` constraint on ShipmentCost, we want the value to be between 0.00 and 999.99, so we'll write our expression as `ShipmentCost` `BETWEEN` `0.00` `AND` `999.99`. Remember that `BETWEEN` includes the beginning and end values, so values of 0.00 and 999.99 are valid. We'll add our `CHECK` constraints much the same way that we added the `DEFAULT` constraint, with our constraint expression included in parentheses:

```sql
CREATE TABLE shipment (
    ShipmentID int NOT NULL,
    OrderID int NOT NULL,
    ShipmentCost decimal(5,2) NOT NULL
        CHECK (ShipmentCost BETWEEN 0.00 AND 999.99),
    ShipmentMethod char(1) NOT NULL CHECK (ShipmentMethod IN ('P', 'E')),
    TrackingNumber varchar(20) NOT NULL UNIQUE,
    ShipmentDate datetime NOT NULL DEFAULT (CURRENT_DATE()),
    CONSTRAINT PK_shipment PRIMARY KEY (ShipmentID),
    CONSTRAINT FK_shipment_orderheader FOREIGN KEY (OrderID)
        REFERENCES orderheader(OrderID)
    );
```

Although the `CHECK` constraints we're creating for the shipment table are fairly simple, as noted earlier, these kinds of constraints can involve multiple columns. We could have written a single constraint with a larger expression to validate that the ShipmentCost was in the stated range of numeric values and that the ShipmentMethod was one of the two acceptable values. When we use multiple columns in the expression of our constraint, we typically want to create the constraint after all the columns, as we did with the `PRIMARY` `KEY` and `FOREIGN` `KEY` constraints.

## 19.2 Automatically incrementing values for a column

We want to make one final change in our table—a very common change that's a bit like a `DEFAULT` constraint. We've established that we want the ShipmentID column to be the primary key of our new shipment table. The primary key values need to be unique so that we can identify individual rows in this table.

In chapter 18, we inserted explicit values for the primary key (CategoryID) of the category table. For many tables, we won't want to use explicit values for the column determined to be the primary key. Instead, we'll want to take advantage of a feature that every RDBMS has—a feature that automatically increments values.

In MySQL, this feature uses the `AUTO_INCREMENT` keyword. Although it's technically not a constraint, it behaves a bit like a `DEFAULT` constraint in that it allows us to insert rows into the table without specifying a value for a column with `AUTO_INCREMENT` enabled.

When we `INSERT` rows into the shipment table with `AUTO_INCREMENT` enabled, we will omit specifying values for the ShipmentID column. The first row inserted this way automatically has a value of 1 for ShipmentID, the second row inserted has a value of 2, and so on. Setting this value to populate the column with incremental values automatically ensures that the primary key will be unique and not NULL.

Also, we'll declare `AUTO_INCREMENT` in our SQL the same way that we've been declaring our constraints. Here's our `CREATE` `TABLE` statement, now with the ShipmentID column set to `AUTO_INCREMENT`:

```sql
CREATE TABLE shipment (
    ShipmentID int NOT NULL AUTO_INCREMENT,
    OrderID int NOT NULL,
    ShipmentCost decimal(5,2) NOT NULL
        CHECK (ShipmentCost BETWEEN 0.00 AND 999.99),
    ShipmentMethod char(1) NOT NULL CHECK (ShipmentMethod IN ('P', 'E')),
    TrackingNumber varchar(20) NOT NULL UNIQUE,
    ShipmentDate datetime NOT NULL DEFAULT (CURRENT_DATE()),
    CONSTRAINT PK_shipment PRIMARY KEY (ShipmentID),
    CONSTRAINT FK_shipment_orderheader FOREIGN KEY (OrderID)
        REFERENCES orderheader(OrderID)
    );
```

Our shipment table, with all the desired constraints, is ready to be created.

##### Try it now

Create the shipment table with the preceding SQL script. You'll practice using this table in the lab exercises at the end of this chapter.

## 19.3 Indexes

Every constraint we've created is intended to ensure the integrity of our data, but in MySQL, some of those constraints also created objects that we haven't looked at yet: indexes. Indexes exist in every RDBMS, so let's take a closer look at them.

An *index* is an object that logically sorts data to make it more readable in commonly used queries, allowing those queries to return data faster. Indexes come in two forms: clustered and nonclustered. We'll look at clustered indexes first.

### 19.3.1 Clustered indexes

*Clustered* indexes are data structures that control the physical order of the rows in a table, which means that their order is how the data will be stored on disks or other storage media. When a table is created without any defined indexes or constraints, the rows of data are stored in no particular order, so any query that uses that table has to read every row to determine whether the values contained should be retrieved, filtered, joined, and so on.

To better understand clustered indexes, think of a telephone book containing the names and telephone numbers of folks who live in your hometown. This telephone book is typically sorted by surname and then the first name of each person, with related telephone numbers and perhaps address information included in each row on any given page. If this telephone book were a table, the clustered index would be on surname and then first name because that order is how the rows are organized.

We want to sort rows based on the most commonly used columns in our table, so we should be thoughtful about the way we create our clustered index. Because the rows in our table can be sorted in only one way, it's important to note that any table can include only one clustered index. After it's created, the clustered index effectively is the table, not a separate object.

If you executed the SQL script in section 19.2, you've already created a clustered index for the shipment table because MySQL created one automatically when you created your `PRIMARY` `KEY` constraint. This situation usually isn't a problem because the clustered index is most often created on the column (or columns) that make up the primary key.

As you may have noticed, when you joined tables in queries, you joined them with related primary and foreign keys. Because the RDBMS needs to read those key values to join rows in different tables, it makes sense to create clustered indexes on the primary-key columns in your tables.

##### Note

Even if you don't define a primary key on a table, the RDBMS creates a hidden column, often called a *row identifier*, that contains a unique value for every row. If you had two or more rows with identical values, the row identifier would allow the RDBMS to know that these rows were different. Because the column is hidden, though, users typically wouldn't see it, so we generally want to create `PRIMARY` `KEY` constraints and clustered indexes for tables that require unique values for each row.

As I said earlier, we already have a clustered index in our table. In MySQL, we can see the indexes in a table in MySQL Workbench. In the Navigator panel, we can expand our sqlnovel database, expand Tables, expand our shipment table, and finally expand Indexes. As shown in figure 19.1, we have three indexes in our shipment table.

##### Figure 19.1 The three indexes of the shipment table in the sqlnovel database, viewed in MySQL Workbench

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH19_F01_Iannucci.png)

We can see even more information about each index by highlighting it and viewing the Information panel, which should be below the Navigator panel. If we highlight the index named PRIMARY, we see the information about this index (figure 19.2).

##### Figure 19.2 Information about the PRIMARY index of the shipment table, viewed in MySQL Workbench

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH19_F02_Iannucci.png)

Although it doesn't explicitly say "clustered index," in MySQL, the index labeled PRIMARY is the clustered index. Admittedly, there isn't much information in the Information panel, but the main consideration for us is the Columns value. This value tells us that ShipmentID is the column used for the clustered index of our table, which is the column we used to define the `PRIMARY` `KEY` constraint.

##### Warning

Because clustered indexes aren't separate objects and are more like properties of a table that are often related to the primary key, every RDBMS has a different way of handling the way they're created. In some RDBMSes, such as SQL Server and DB2, you can create them explicitly using an `ALTER` `TABLE` or `CREATE` `INDEX` statement, but in others, such as MySQL and PostgreSQL, you can't. Refer to the documentation for your specific RDBMS to see what options you have for creating clustered indexes.

### 19.3.2 Nonclustered indexes

Although clustered indexes are common and highly beneficial to performance, nonclustered indexes are also helpful in speeding our queries. A *nonclustered* index is different from a clustered index in two main ways:

* Unlike clustered indexes, nonclustered indexes are separate objects from the tables they relate to.
* Because nonclustered indexes are separate objects, a table can contain more than one of them.

A good analogy for a nonclustered index is a catalog system for books in a library. Nonfiction books in a library are stored under a numeric catalog system, such as the Dewey Decimal System. Most of us don't look for a book by its numeric value in this system, however; rather, we use the title. We use the catalog to look up the title of our desired book, which gives us the numeric value of the book we want. Then we go through the shelves of the library that are ordered by the numeric system and find our book.

Nonclustered indexes work like the catalog system in a library. They're a separate ordering of the books, in this case by title, which allows us to quickly find the title and numeric value and then use that value to go to the place in the library where the book exists. In this analogy, the numeric value would be the primary key and clustered index of the nonfiction books in the library.

As you can see from this analogy, the catalog system (the nonclustered index) greatly speeds the search for the book we're looking for. If we didn't have the catalog, we'd have to scan the entire library until we found the book. We create nonclustered indexes for the same reason: to improve performance in finding rows of data in a table without reading the entire table.

Although we can put all the columns we want in a nonclustered index, we generally have only a few columns or even a single column. Typically, we're searching only on one or two columns, so we don't want to make our nonclustered indexes larger than necessary. The more columns the indexes have, the more storage space they take up and the more resources they require for any `INSERT`, `UPDATE`, and `DELETE` statements involving the table they relate to.

We saw in figure 19.1 that we have two nonclustered indexes in our shipment table, so let's examine them. To look at them, we can click them in the Navigator panel and review their information in the Information panel. Let's look at the TrackingNumber index information, shown in figure 19.3.

##### Figure 19.3 Information about the TrackingNumber index of the shipment table, viewed in MySQL Workbench

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH19_F03_Iannucci.png)

Because we created a `UNIQUE` constraint on the TrackingNumber column, MySQL automatically created a nonclustered index on this column. As figure 19.3 indicates, this index has a value of Yes for the Unique property.

MySQL is unusual in that it created the index for this column automatically; many other RDBMSes wouldn't do that. Because this column is required to contain unique values, we likely would want to create a nonclustered index for the column anyway.

If we think about the nature of this column, which contains tracking numbers for shipments, we expect queries to look for shipment information related to one specific TrackingNumber. Rather than scan the entire shipment table every time we want to find data related to a specific TrackingNumber, our queries can use this nonclustered index. This is exactly the kind of column we would want a nonclustered index on.

The analogy earlier in this chapter discusses a library catalog that works as a sort of nonclustered index for all nonfiction books, and here, the TrackingNumber would serve the same purpose for the tracking numbers of our shipments. If this index on TrackingNumber hadn't been created automatically (and it won't be in many other RDBMSes), we could create it with the following SQL statement:

```sql
CREATE INDEX IX_shipment_TrackingNumber ON shipment (TrackingNumber);
```

This syntax is common to just about every RDBMS, so I don't have to add any qualifiers about its use.

##### Tip

The preceding SQL statement that creates the nonclustered index on the shipment table uses a common but specific naming convention: IX to indicate an index, an underscore, the table name, another underscore, and the name of the column of the index. As always, be intentional about the names of objects in your database, and follow a consistent naming convention to make objects that other people can easily understand.

Figure 19.1 indicates we have a third index in our table: FK_shipment_orderheader. This index was created automatically by our `FOREIGN` `KEY` constraint, which is a behavior of MySQL but not necessarily a behavior of other RDBMSes. It's common to create nonclustered indexes on the columns contained in `FOREIGN` `KEY` constraints, however, because these columns are used to link tables via the relationships in the keys. Having the nonclustered indexes on the foreign key columns can reduce the need to read all the data in a table to join with other tables. We can see the properties of this index by clicking FK_shipment_orderheader in the Navigator panel and then viewing the Information panel (figure 19.4).

##### Figure 19.4 Information about the FK_shipment_orderheader index of the shipment table, viewed in MySQL Workbench

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH19_F04_Iannucci.png)

One thing to note about this index: the value of the Unique property is No, which is different from the other two indexes of our shipment table. Although the `UNIQUE` constraint we made also created an index that requires unique values, it's important to note that nonclustered indexes aren't required to have unique values. In the case of the FK_shipment_orderheader index, unique values aren't required because we might have a one-to-many relationship between the orderheader and shipment tables as far as OrderID values are concerned.

This chapter covered a lot of database design concepts. Let's briefly summarize the main points about constraints and indexes:

* Constraints are properties on one or more columns that enforce data integrity.
* `NOT` `NULL` constraints ensure that no NULL values are contained in a column.
* `DEFAULT` constraints enter a default value if no value is specified for a column on `INSERT`.
* `UNIQUE` constraints enforce that all values in a column are different.
* `CHECK` constraints are used to limit the range of values that can be contained in a column.
* A clustered index defines the physical sort order of a table, typically on the primary key.
* A table can have only one clustered index.
* A nonclustered index is a separate object from the table.
* A table can have many nonclustered indexes, although every additional nonclustered index negatively affects the performance of `INSERT`, `UPDATE`, and `DELETE` statements.
* In MySQL (but not every RDBMS), a clustered index is created automatically when we define a `PRIMARY` `KEY` constraint.
* In MySQL (but not every RDBMS), a nonclustered index is created for every `UNIQUE` or `FOREIGN` `KEY` constraint we create.

If you're feeling up to it, try to flex your new skills with constraints and indexes in the lab exercises.

## 19.4 Lab

1.  Using the following values, write a SQL statement to insert rows into the new shipment table:

•   `OrderID` = `1001`

•   `ShipmentCost` = `0.00`

•   `ShipmentMethod` = `'P'`

•   `TrackingNumber` = '`1A2C3M4E'`

2.  Because the shipment table currently doesn't have a row for every order, how would you write a query to see the OrderID, OrderDate, and ShipmentDate for every order?

3.  Could you have written the expression in section 19.1.4 differently? If so, how?

4.  You want to create a report that shows the count of all orders shipped on a particular date. What constraint or index could you create to improve the performance of this report?

## 19.5 Lab answers

1.  You don't have to specify a value for ShipmentID because it's an `AUTO_INCREMENT` column, and you don't have to specify a value for ShipmentDate because it has a default constraint. Therefore, your SQL should look something like this:

```sql
INSERT shipment (
    OrderId,
    ShipmentCost,
    ShipmentMethod,
    TrackingNumber
    )
VALUES (
    1001,
    0.00,
    'P',
    '1A2C3M4E'
    );
```

2.  Because you have values for every OrderID in orderheader but don't have values for every OrderID in shipment, you need to use a `LEFT OUTER JOIN`, like this:

```sql
SELECT
    oh.OrderID,
    oh.OrderDate,
    s.ShipmentDate
FROM orderheader oh
LEFT OUTER JOIN shipment s
    ON oh.OrderID = s.OrderID;
```

If you used an `INNER` `JOIN` instead, your result set would include only orders that have a value in the OrderID column of both tables.

3.  You could write this expression in a few ways, including this way:

```sql
 ShipmentCost >= 0.00 AND ShipmentCost <= 999.99.
```

4.  The SQL used in your report would look something like this:

```sql
SELECT
    ShipmentDate,
    COUNT(ShipmentDate)
FROM shipment
WHERE ShipmentDate = @ShipmentDate
GROUP BY ShipmentDate;
```

To support this query, you'd create a nonclustered index on the ShipmentDate column:

```sql
CREATE INDEX IX_shipment_ShipmentDate ON shipment (ShipmentDate);
```

This nonclustered index would help keep you from having to read the entire table to determine the total of what would be a fraction of the orders shipped on any given day.

