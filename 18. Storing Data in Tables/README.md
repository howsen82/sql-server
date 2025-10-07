# 18 Storing Data in Tables

In chapters 16 and 17, we started creating and manipulating data, and in this chapter, we'll examine ways to create and manipulate the tables themselves. In many ways, we'll be getting to the core of SQL language and its use because how the data is stored in tables is at the very heart of any relational database management system (RDBMS). Choosing how data is stored is one of the most important—perhaps *the* most important decision—that is made in any database.

Don't worry; this chapter won't be overly technical. It will still be easy enough for anyone to understand, and because you've been querying data with SQL statements that often mirror the English language for a while now, I'm confident that you'll find the concepts and commands easy to comprehend. This chapter should also reinforce your understanding of things like primary keys and data types.

## 18.1 Creating a table

As you'll soon see, creating a table in SQL can be very simple. First, though, you must consider a few things about the table before you write the SQL that creates the table.

### 18.1.1 Considerations before creating a table

The first step in creating a table is answering three basic questions about the table:

* What is the name of the table?
* What are the names of the columns that will be included in the table?
* What are the data types of those columns?

We'll have to put some thought into these names and data types, especially as they relate to existing tables in the database. We want the names to be explanatory, but we can't use a table name if there's already a table with that name.

We can reuse column names from other tables, however, as we've seen with the sqlnovel database: OrderID, TitleID, and other column names appear in more than one table. But we usually want to do that only if some relationship exists between similarly named columns.

##### Tip

The idea of having similar names is one reason why I've used names such as OrderID and TitleID in the sqlnovel database. One day, you may work with a database containing many tables that have columns named ID or Name, which can be a bit confusing due to ambiguity. As you create tables, try to make column names meaningful, clear, obvious, and easy to understand for anyone else who may have to query the data.

As an exercise, we want to create a table for categories of the titles. Although we could avoid creating a new table by adding a Category column in the title table for a string of characters such as "Mystery" or "Romance," we know from the discussion of table relationships in chapter 8 that this approach isn't the best one to take in an RDBMS. (Later in this chapter, we'll add a column to the title table, but that column will relate to the primary key of a new table.)

For our new table of categories, we need to have only two columns: an ID column to serve as the primary key and a column to define the name of each category. We can keep the table name and columns consistent with the rest of our database by naming the table category and the columns CategoryID and CategoryName.

The ID column of a table is often defined as an integer data type, which is known as `int`. The `int` data type allows us to use a unique number for each row—usually up to numbers in the billions. I don't think this table will have billions of categories, so an integer data type should accommodate CategoryID. We could use some other integer data types—such as `tinyint`, `smallint`, `mediumint`, and `bigint`—but because they aren't included in every RDBMS, we'll use the universal `int` data type.

Columns with name values of varying length, such as CategoryName, are typically defined as a variable character type, known as `varchar.` This data type accounts for the fact that not every value will be the same length (number of characters). When we use this data type, our data is stored more efficiently than it would be if we'd used a `char` (character) data type because that data type stores the data using the entire defined length.

Although defining the maximum length isn't required for `varchar` data types, we typically want to do this to avoid using an inefficient default value, which will vary depending on our RDBMS. The values in our CategoryName column won't be more than 20 characters, so we'll define the data type as `varchar(20)`.

##### Note

As you converse with other people about SQL, you'll discover that there's no common way to pronounce *varchar*. Pronunciations vary from "var-char" to "var-kar" to "vair-kair." The last option is most likely to be correct because the vowels match the pronunciation of the first syllables in *variable character*, but I've found that it's also the least likely to be used. Try not to get too confused by this situation. As the French say, *vive la différence*.

### 18.1.2 Creating a table

Now that we've defined our table name, column names, and column data types, let's say in English what we intend to do: "I would like to create a table named category. I would like the table to have a column named CategoryID that is an int data type. I would like the table to also have a column named CategoryName that is a varchar(20) data type." After all that, here's what the SQL to create our category table will look like:

```sql
CREATE TABLE category (
    CategoryID int,
    CategoryName varchar(20)
    );
```

Notice that after we define the table name with `CREATE` `TABLE`, we include all columns with a comma separating the names and data types of each column. This format should be a bit intuitive now that we've used commas to separate columns in `SELECT` and `ORDER` `BY` clauses. Also notice that the columns are enclosed in parentheses. Omitting the parentheses when using `CREATE` `TABLE` can be a common mistake for beginners, so remember to include them when creating a table.

Executing the preceding SQL won't return any results in the Results panel, but in the Output panel, you should see a green circle with a white check mark and the message "0 rows(s) affected." Although the panel doesn't show a lot of detail, it tells you that the table was created successfully.

Creating a table in this way is just a basic starting point. As we'll see later in this chapter and in subsequent chapters, the `CREATE` `TABLE` statement gives us quite a few options for adding more properties to a table. For now, we'll move to the next step, using what we learned about the `INSERT` keyword in chapter 16.

### 18.1.3 Adding values to an empty table

Our customer table is empty, so next, we want to insert the CategoryID and CategoryName values for the following categories:

1. Romance
2. Humor
3. Mystery
4. Fantasy
5. Science Fiction

Chapter 16 discussed adding multiple values to a table by using the `INSERT` and `VALUES` keywords. Let's use similar logic to insert these listed values into our new category table:

```sql
INSERT INTO category (CategoryID, CategoryName)
VALUES
    (1, 'Romance'),
    (2, 'Humor'),
    (3, 'Mystery'),
    (4, 'Fantasy'),
    (5, 'Science Fiction');
```

Executing the preceding SQL won't return anything in the Results panel, but if the execution is successful, we should see "5 row(s) affected" in the Message column of the Output panel. This message indicates that we added five rows to our category table, which was our intention.

##### Try it now

If you haven't created and populated the category table yet, execute the SQL in sections 18.1.2 and 18.1.3 to do so. You'll be using this table in this chapter and in subsequent chapters.

We can verify that we added the desired values to the category table by running a simple `SELECT` query to see the values in the table (results shown in figure 18.1):

```sql
SELECT
    CategoryID,
    CategoryName
FROM category;
```

##### Figure 18.1 All rows in the new category table

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH18_F01_Iannucci.png)

## 18.2 Altering a table

The next step in adding a category for each title is adding a new column to the title table that relates to the values in our new category table. Specifically, we want to relate the CategoryID values from the title table to those in the category table. We'll do this by adding a CategoryID column to our title table.

### 18.2.1 Adding a column to a table

Just as we had to consider three questions before creating a table, we need to consider three questions before adding a column:

* What is the name of the table we are adding the new column to?
* What are the names of the columns that will be added to the table?
* What are the data types of those columns?

We know that the answer to the first question is the title table. Because we want consistency in names and data types, the answers to the second and third questions are related to the CategoryID column in the category table: CategoryID and `int`, respectively.

To make these and other changes in a table in SQL, we'll use a new command: `ALTER` `TABLE`. Although the syntax of this command is similar to that of `CREATE` `TABLE`, it doesn't require us to use parentheses. Here's the SQL to add the column to the title table:

```sql
ALTER TABLE title
ADD CategoryID int;
```

As with our `CREATE` `TABLE` statement in section 18.1.2, we won't have any results from this query. The success of this query is noted in the Output panel only by a white check mark in a green circle and the message "0 row(s) affected."

Let's run a quick query to validate that the column was created, noting the results in figure 18.2 that show the column was created after all the other columns:

```sql
SELECT *
FROM title;
```

##### Figure 18.2 The title table with the new CategoryID column at far right

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH18_F02_Iannucci.png)

Our new column doesn't contain any values yet, so the results of our query show NULL for every row. To add the values, we'll use `UPDATE` statements. Although we used `INSERT` to add values to our category table, `INSERT` adds a new row to a table. We don't want to do that here; we want to add a value for a single column to the existing rows, which `UPDATE` allows.

Let's add these values for all rows based on the category of each title. In case you didn't memorize all the CategoryID and CategoryName values or don't feel like flipping back a few pages, the following SQL has comments to remind you:

```sql
/* 1 – Romance */
UPDATE title
SET CategoryID = 1
WHERE TitleID IN (101, 104);

/* 2 – Humor */
UPDATE title
SET CategoryID = 2
WHERE TitleID IN (106, 109);

/* 3 – Mystery */
UPDATE title
SET CategoryID = 3
WHERE TitleID IN (102, 103, 110);

/* 4 – Fantasy */
UPDATE title
SET CategoryID = 4
WHERE TitleID IN (107, 112);

/* 5 – Science Fiction */
UPDATE title
SET CategoryID = 5
WHERE TitleID IN (105, 108, 111);
```

After we execute all the preceding `UPDATE` statements, we should see values for the CategoryID column in all rows. Let's execute our query to return all rows of the title table, verifying in figure 18.3 that all values for CategoryID are now populated:

```sql
SELECT *
FROM title;
```

##### Figure 18.3 The title table with the CategoryID populated with values for all rows

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH18_F03_Iannucci.png)

We've confirmed that we added the values for CategoryID, but those values don't tell us directly what the category names for each title are. Let's show the CategoryName for each TitleName by writing a query that relates the title and category tables by CategoryID. This relationship is established with an `INNER` `JOIN` (discussed in chapter 8). The results in figure 18.4 confirm the category for each title, ordered by TitleID:

```sql
SELECT
    t.TitleID,
    t.TitleName,
    c.CategoryName
FROM title t
INNER JOIN category c
    ON t.CategoryID = c.CategoryID
ORDER BY t.TitleID;
```

##### Figure 18.4 The CategoryName for every TitleName in the title table, related by CategoryID

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH18_F04_Iannucci.png)

### 18.2.2 Considerations before adding a column

Adding a category for each title can be relatively simple, but as with creating a table, we have a few things to consider before adding new columns to tables.

When a column is added to a table, a SQL novice may be bothered by the fact that the new column is added at the end of all the other columns. A new column is always added at the end, although some RDBMSes (such as MySQL) allow us to change the order of the columns. This change, however, is generally discouraged for tables that already contain data.

There are two main reasons for adding columns only at the end of all other columns:

* Adding a new column before any other columns requires a lot of extra activity to rearrange the data, using more resources than adding the column at the end. When we add the column at the end, we minimize the amount of activity and resources that will be required.
* Generally, it's unnecessary to position columns in a certain order for a table. As we've already seen, we can order columns for display however we like in our SQL queries.

The one exception to adding a column at the end occurs when we create a new column to be used in the primary key of a table. We generally want that column to come first because the primary key often defines the way that data is ordered in a table. Although I've referred to primary keys since chapter 8, section 18.3 takes a deeper look at how to create and use them.

## 18.3 Primary keys

This book is designed to get you up to speed gradually with writing SQL queries, so I haven't discussed primary keys much. But because now I'm talking about designing tables and relating data, it's time to revisit that topic.

### 18.3.1 Considerations for primary keys

The primary key is the backbone of any RDBMS because it ensures that the data in tables is relatable. With primary keys, we have to adhere to a few rules:

* *A primary key must be unique.* This rule is the most important one. If the primary key isn't unique, we'll never know which row in a relationship is the correct one. Suppose that we assigned a value of 1 to the CategoryID column for each of the five rows in our new category table. If all five rows had the same value, we couldn't possibly determine the correct CategoryName for any relationship in which the CategoryID is 1.
* *Every row must have a value for the primary key.* As we've seen, we can't join a value of NULL to any value, including other values of NULL. For this reason, if any row in a table has NULL as a primary-key value, that row can never be related to any other data.
* *The primary key's values can never change.* We use primary key values to relate to other tables, much as we used CategoryID values to relate to the title table. If we changed the CategoryID values in the category table in any way, such as by adding 10 to the existing value, the values in the two tables wouldn't match, and the relationship between the two tables would no longer exist.

Adhering to these three rules allows relationships that use primary keys to maintain *referential integrity*, which means that any value from one table that refers to the primary key value in another table will always refer to the same value in the table that contains the primary key. Although the values for other columns in our table can change (we could change a CategoryName from Mystery to Mystery Thriller, for example), the values for the primary key can never change.

### 18.3.2 Adding a primary key

Now that we know the rules for any primary key, we can create one for our category table. To do this, we can use the same `ALTER` `TABLE` statement that we used to add a column, with some slight differences:

```sql
ALTER TABLE category
ADD CONSTRAINT PRIMARY KEY (CategoryID);
```

There are two notable differences between adding a column and adding a primary key with `ALTER` `TABLE`. First, we used the word `CONSTRAINT` when noting that we were adding a primary key. I haven't explicitly said it before, but a primary key is a kind of *constraint*—an object designed to enforce a rule of some sort. Therefore, we can't break the rules of the constraint after it's created. With this new `PRIMARY` `KEY` constraint, we must adhere to the rules concerning the category table outlined in section 18.3.1.

Although doing so isn't required, it's common to give a primary key a logical name. The reason is that you may create other objects in your table that have different constraints, and you'll want the names of these objects to indicate their purpose. Just as we give tables obvious names for their data, such as customer and author, we want to give our primary keys obvious names. Moreover, as you'll see later in this chapter, assigning a name to a primary key (or any other kind of constraint) makes it much easier to manipulate if you have to change or delete it later.

A common way to name primary keys is to use the naming convention `PK_`, which is the `PK` prefix followed by an underscore and the table name. Even if you know nothing about a particular database, if you saw any reference to an object named `PK_category`, you'd understand it to be the `PRIMARY` `KEY` constraint of a table named category. For this reason, it's desirable to create our primary key with a name:

```sql
ALTER TABLE category
ADD CONSTRAINT PK_category PRIMARY KEY (CategoryID);
```

##### Try it now

Create the `PRIMARY` `KEY` constraint for the category table with the preceding SQL statement.

Although here, you're adding the primary key to an existing table, most of the time, you'll add the primary key when you create the table. You can accomplish this task easily by defining the primary key in the `CREATE` `TABLE` statement after defining the columns, almost as though you were adding another column. You could have created the category table with the primary key defined by using the following SQL:

```sql
CREATE TABLE category (
    CategoryID int,
    CategoryName varchar(20),
    CONSTRAINT PK_category PRIMARY KEY (CategoryID)
    );
```

##### Tip

Although it isn't necessary to create a primary key for every table in every database, a primary key belongs in any table that has (for lack of a better phrase) a group of unique entities. Tables with rows that represent unique entities such as products, orders, and customers should always have a primary key to ensure that those rows are unique.

One final note about primary keys: they can consist of more than one column. Our orderitem table, for example, should have a primary key that includes both OrderID and ItemID because together, these columns form a unique key. Multiple rows may have the same OrderID, but each of those rows should also have a unique ItemID for every row that includes the same OrderID.

To create a primary key with multiple columns in a table such as orderitem, we'd use a comma separator when specifying the columns, like this:

```sql
ALTER TABLE orderitem
ADD CONSTRAINT PK_orderitem PRIMARY KEY (OrderID, OrderItem);
```

Using primary keys allows us to ensure that we can identify each row in a table as unique. Because these rows are often referred to by other tables in relationships, we can explicitly define these relationships with another kind of constraint: the foreign key.

## 18.4 Foreign keys and constraints

*Foreign keys* are established to enforce the rules of any relationship between two tables with respect to a common column. Any foreign-key relationship involves a *parent table*, where the key values originate, and *child tables*, which contain values that refer to the parent table. Put simply, we create a foreign key on a child table to enforce the rule that the values in the child table must exist in the parent table.

### 18.4.1 Data diagrams

*Data diagrams* can help us understand the relationships between tables in our database by depicting these relationships visually. Figure 18.5 is a data diagram of all the tables in the sqlnovel database, including our new category table.

##### Figure 18.5 A data diagram that shows all tables in the sqlnovel database and their relationships to other tables

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH18_F05_Iannucci.png)

Each box in this diagram represents a table, and each box contains a list of all columns and data types for that table. We want to focus on the lines between the boxes, which represent the relationships. If a line between two tables exists, a foreign key from one table to another exists.

Although the data diagram in figure 18.5 doesn't show the specific columns involved in the relationships, you can find out what they are by examining the tables closely. If you follow through and execute the prescribed exercises throughout this chapter and in the lab section, you'll see how to create your own data diagram.

One other thing to note: the myfirstquery table has no lines connecting it to other tables. That table has no relationship to any other tables in the database because it was used only to get you started with your first query in chapter 2. You've come a long way since then!

### 18.4.2 Adding a foreign-key constraint

Because a functional foreign key requires another type of constraint, we'll use an `ALTER TABLE` statement similar to the one we used to create the `PRIMARY` `KEY` constraint. As with the primary key, we want to give our `FOREIGN` `KEY` constraint a logical name. A common approach is to use a name in the format `FK_`, which is the `FK` prefix followed by an underscore, the name of the child table, another underscore, and the name of the parent table. This format ensures that the name of our `FOREIGN KEY` constraint will be unique in the database. Using this naming convention, we can create our `FOREIGN KEY` constraint with the following statement:

```sql
ALTER TABLE title
ADD CONSTRAINT FK_title_category
FOREIGN KEY (CategoryID) REFERENCES category(CategoryID);
```

##### Try it now

Create the `FOREIGN` `KEY` constraint on the title table with the preceding SQL statement.

This statement is a bit different from the one we used to create the `PRIMARY KEY` constraint because we're creating a key relationship between two columns in different tables. The first column, after the keywords `FOREIGN KEY`, indicates the column on the child table that will reference another column in the parent table, which is why we use the `REFERENCES` keyword.

We want to be careful when we create constraints—`PRIMARY` `KEY`, `FOREIGN` `KEY`, or otherwise—on tables that already contain data because if the current values don't meet the rules of our constraint, we'll get an error. For this reason, it's best to create constraints on tables when we create the table and before we insert any rows of data.

##### Tip

Although this chapter uses common conventions to name constraints, these aren't the only ways to name constraints. When you work outside the sqlnovel database, consider whether the database you're working with already uses defined naming conventions. If so, create your objects using the existing naming conventions so that your object names are consistent with the names of other objects in the database.

## 18.5 Deleting a table, column, or constraint

Although we want to keep the objects we've created in this chapter, if we want to undo our work, we can do so with statements that use the `DROP` keyword.

##### Warning

The SQL in this section is provided for informational purposes only. You don't want to drop the category table or any of its related columns and constraints because you'll be using this data throughout the remainder of the book. If you decide that you want to practice dropping these objects despite this warning, you'll have to go back through this chapter to re-create them.

With that warning out of the way, here's how we could remove the objects we have created in this chapter.

### 18.5.1 Deleting a constraint

As with adding a constraint, deleting any constraint involves the `ALTER` `TABLE` statement. Because we're dropping our constraint, not defining anything, we need only the names of the table and the constraint that we're dropping. In this case, we'd use the following SQL:

```sql
ALTER TABLE title
DROP FOREIGN KEY FK_title_category;
```

If we want to delete the `PK_category` primary key of our category table, we could do that in either of two ways. The first way is to remove it as a constraint, like this:

```sql
ALTER TABLE category
DROP CONSTRAINT PK_category;
```

Because the primary key is a special kind of constraint, however, we could use `ALTER TABLE` to say that we want to remove the primary key without supplying the name of the constraint:

```sql
ALTER TABLE category
DROP PRIMARY KEY;
```

##### Note

We don't need to specify the name of the primary key in the statement because the table can have only one primary key.

### 18.5.2 Deleting a column

In this chapter, we created a column in the title table. To remove that column, we'd use `ALTER` `TABLE` `and` `DROP:`

```sql
ALTER TABLE title
DROP COLUMN CategoryID;
```

As with dropping constraints, we typically don't need more than the names of the table and column to remove a column.

### 18.5.3 Deleting a table

Deleting a table involves the least SQL of any of our object-removal scripts. We use `DROP TABLE` with the table name:

```sql
DROP TABLE category;
```

##### Note

If we want to remove all these objects, we'd need to do it in the order in section 18.5, removing the constraints first. If we want to drop the table or column first, most RDBMSes (including MySQL) would return an error message saying that the object can't be dropped because of these constraints.

That's enough discussion of removing objects. Let's start the lab, where we can practice creating constraints such as primary keys and foreign keys.

## 18.6 Lab

1.  The sqlnovel database is missing a few primary keys. Using the data diagram in section 18.4.1 and the naming conventions in section 18.3.2, write and execute SQL statements to add `PRIMARY` `KEY` constraints for the following tables:

•   author

•   customer

•   orderheader

•   promotion

•   title

•   titleauthor

2.  The orderitem table isn't included in the preceding list. What happen if you try to create a `PRIMARY` `KEY` constraint on the OrderID and OrderItem columns? How can you resolve this problem?

3.  The sqlnovel database is also missing a few `FOREIGN` `KEY` constraints. Using the data diagram in section 18.4.1 and the naming conventions in section 18.4.2, write and execute SQL statements to add `FOREIGN` `KEY` constraints for the following tables and columns:

•   The CustomerID column of the orderheader table

•   The PromotionID column of the orderheader table

•   The OrderID column of the orderitem table

•   The TitleID column of the orderitem table

•   The TitleID column of the titleauthor table

•   The AuthorID column of the titleauthor table

4.  If you've successfully completed all the preceding tasks, now is your chance to enjoy your work. Create a data diagram. In MySQL Workbench, choose Database > Reverse Engineer. Click Next in all the following screens, and be sure to select the sqlnovel check box in the Select Schemas screen. When you're done, you should have a data diagram of the sqlnovel database.

## 18.7 Lab answers

1.  You can create `PRIMARY` `KEY` constraints for these tables with the following SQL statements:

```sql
ALTER TABLE author
    ADD CONSTRAINT PK_author PRIMARY KEY (AuthorID);
ALTER TABLE customer
    ADD CONSTRAINT PK_customer PRIMARY KEY (CustomerID);
ALTER TABLE orderheader
    ADD CONSTRAINT PK_orderheader PRIMARY KEY (OrderID);
ALTER TABLE promotion
    ADD CONSTRAINT PK_promotion PRIMARY KEY (PromotionID);
ALTER TABLE title
    ADD CONSTRAINT PK_title PRIMARY KEY (TitleID);
ALTER TABLE titleauthor
    ADD CONSTRAINT PK_titleauthor PRIMARY KEY (TitleID, AuthorID);
```

2.  The statement that creates the `PRIMARY` `KEY` constraint on orderitem looks like this:

```sql
ALTER TABLE orderitem
    ADD CONSTRAINT PK_orderitem PRIMARY KEY (OrderID, ItemID);
```

If you execute this statement, however, the Output window displays the error message "Error Code: 1062. Duplicate entry '1022-1' for key 'orderitem.PRIMARY'." This error message indicates a data inconsistency in what would be your primary key, and it tells you where that error is. The error is for OrderID 1022 and OrderItem 1. You can see the problem by executing the following query, which should return one row but instead returns two rows (figure 18.6):

```sql
SELECT *
FROM orderitem
WHERE OrderID = 1022
    AND OrderItem = 1;
```

##### Figure 18.6 The two rows that prevent the primary key from being created for the orderitem table

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH18_F06_Iannucci.png)

There are two rows that would result in duplicate values for our primary key, which isn't allowed. All key values must be unique. Fortunately, these rows don't appear to be actual duplicates because they have different TitleID values. You can safely correct this data error with an `UPDATE` statement, changing the OrderItem value from 1 to 2 for one of the rows:

```sql
UPDATE orderitem
SET OrderItem = 2
WHERE OrderID = 1022
    AND OrderItem = 1
    AND TitleID = 103;
```

After executing the preceding `UPDATE` statement, you should be able to create the primary key for the orderitem table. Create that `PRIMARY` `KEY` constraint.

3.  You can create the `FOREIGN` `KEY` constraints for these tables and columns with the following SQL statements:

```sql
ALTER TABLE orderheader
    ADD CONSTRAINT FK_orderheader_customer FOREIGN KEY (CustomerID)
    REFERENCES customer(CustomerID);
ALTER TABLE orderheader
    ADD CONSTRAINT FK_orderheader_promotion FOREIGN KEY (PromotionID)
    REFERENCES promotion(PromotionID);
ALTER TABLE orderitem
    ADD CONSTRAINT FK_orderitem_orderheader FOREIGN KEY (OrderID)
    REFERENCES orderheader(OrderID);
ALTER TABLE orderitem
    ADD CONSTRAINT FK_orderitem_title FOREIGN KEY (TitleID)
    REFERENCES title(TitleID);
ALTER TABLE titleauthor
    ADD CONSTRAINT FK_titleauthor_title FOREIGN KEY (TitleID)
    REFERENCES title(TitleID);
ALTER TABLE titleauthor
    ADD CONSTRAINT FK_titleauthor_author FOREIGN KEY (AuthorID)
    REFERENCES author(AuthorID);
```

4.  There's no correct answer if you created the data diagram successfully. Have fun moving around the tables to make the lines representing the relationships clearer, and be sure to hover over the lines to see how they highlight the columns represented in the relationships.

