# 3 Querying Data

In chapter 2, we looked at a spreadsheet of fictional books to better understand some core concepts about tables in a relational database management system (RDBMS). With that spreadsheet in mind, we're going to work with a table that looks a lot like that spreadsheet and see some of the ways we can retrieve data using the `SELECT` command.

First, though, let's take a deeper look at your first query. Although it was simple, it had all the minimum components for a query. Let's briefly examine those components as well as some potential problems regarding formatting and the use of certain words.

## 3.1 Rules for the SELECT statement

Chapters 1 and 2 discussed the conversational way to think about writing a query, so let's take a moment to consider the technical aspects and requirements as well. Recall your first query, which looked like this:

```sql
SELECT Outcome FROM MyFirstQuery;
```

This statement has four components, each represented by a single word. Technically, the semicolon is a component as well, serving as the statement terminator, but we've already discussed the fact that it may not be required, so we won't count it.

### 3.1.1 SELECT requirements

The words "Outcome" and "MyFirstQuery" reflect the data we want to select. These words are crucial because they provide the minimum information the database needs to retrieve data from a table. These requirements are

* What data is to be selected
* Where the data is to be selected

In this case, the data to be selected is the Outcome column, and the location where the data is to be selected is the MyFirstQuery table. Both of those words follow specific keywords that are included in the SQL catalog of commands: `SELECT` and `FROM`, each of which represents a *clause* in your SQL statement. All SQL statements are made up of various clauses, but to retrieve data from a table, we're required to use at least these two. We commonly refer to clauses by the keywords used in them, so these two would be called the SELECT clause and the FROM clause.

##### NOTE

We always identify the data we want to select immediately after the `SELECT` keyword, and we always indicate the location where we want to select data after the `FROM` keyword.

It's important to note the order in which these clauses are used in SQL. Throughout the book, you'll learn several more clauses, and they must always be used in a particular order for a query to execute. As an example, we couldn't successfully switch the order of clauses in your first query:

```sql
FROM MyFirstQuery SELECT Outcome;
```

Attempting to execute this query would result in a syntax error because the SELECT clause must always come before the FROM clause.

### 3.1.2 Keywords and reserved words

Keywords such as `SELECT` and `FROM` are a subset of *reserved words* in the SQL language used by each RDBMS. When the RDBMS you're using finds those reserved words in a query, it presumes that you want it to complete a specific action associated with the reserved word. Numerous reserved words are universal, but some reserved words are specific to the RDBMS you're using.

As you progress in your SQL knowledge, take note of reserved words you use as commands so that you'll know not to use them as table names or column names.

##### TIP

If you want to know all the reserved words, you can find them in the documentation on the site where you downloaded MySQL and MySQL Workbench. Every major RDBMS has online documentation that catalogs its specific reserved words. As a general rule, if you're working in a development interface like MySQL Workbench, you'll see reserved words in a different color from the rest of your SQL.

Using reserved words for object names results in avoidable headaches caused by syntax errors because the RDBMS will be confused about what you want to do. Suppose that the MyFirstQuery table had a column named Select, and you wanted to execute the following query:

```sql
SELECT Select FROM MyFirstQuery;
```

##### Try it now

Type this SQL in your Month of Lunches connection in MySQL Workbench. You'll see the word `Select` in a different color from `MyFirstQuery`, which is your first clue that `Select` is a reserved word. As noted earlier, each RDBMS has dozens or even hundreds of reserved words, so when you see the color indicated for a reserved word, take caution before executing your query.

If you execute the preceding query, you'll get the error message that says, "You have an error in your SQL syntax." The MySQL database engine saw the reserved word `Select` consecutively, which won't work because you never said what you wanted to select after the first time you said `SELECT`.

### 3.1.3 Case insensitivity

While we're looking at this query that won't execute, notice that both `SELECT` and `Select` are identified as reserved words, even though they're in a different case. Keywords aren't case-sensitive, so each of the following queries will successfully execute and return the same result:

```sql
SELECT Outcome FROM MyFirstQuery;
Select Outcome From MyFirstQuery;
select Outcome from MyFirstQuery;
SeLeCt Outcome fRoM MyFirstQuery;
```

Just because you can use any kind of case with your keywords, however, doesn't mean you should. To write reusable SQL, many developers prefer typing keywords in uppercase to make code more readable and therefore make it easier to debug errors. For this reason, the examples throughout this book will continue to show keywords in uppercase.

##### WARNING

Although SQL keywords can be used without regard to case sensitivity, the information relating to data may be case-sensitive, depending on the settings of your RDBMS. Be careful when specifying table, column, or value names in your queries because they may be case-sensitive.

### 3.1.4 Formatting and whitespace

One other thing to note about queries is the flexibility you have when using whitespace. Your RDBMS doesn't care much about it, so you can format your query in a nearly infinite number of ways with spaces, tabs, and carriage returns. Your first query was written on a single line, but it would work the same way if it were separated into several lines. All three of the following queries, for example, will execute and return the same result.

##### Query 1

```sql
SELECT Outcome
FROM MyFirstQuery;
```

##### Query 2

```sql
SELECT
Outcome
FROM
MyFirstQuery;
```

##### Query 3

```sql
SELECT
       Outcome
FROM
       MyFirstQuery;
```

Although there are no universal best practices in formatting, the best advice I can give you is to be consistent. The goal of adjusting the format is to make the query more readable, so if you find a particular way of formatting that's easy for you to work with, use it and stick with it.

I think we've gotten all we can out of your first query. It's time to move on to querying data that may be a bit more comparable to the kind you'll need to work with.

## 3.2 Retrieving data from a table

For the rest of this chapter, we're going to examine the title table in our sqlnovel database. Unlike the MyFirstQuery table, the title table has several columns and multiple rows of data.

Unless you examined the scripts used to create this database, you probably don't know the names of the columns in the title table. Fortunately, we can find this information easily by using MySQL Workbench. Look at the top-left corner of the Navigator panel, and notice the triangles next to sqlnovel and Tables. The triangle next to sqlnovel points down, which indicates that it has been expanded. This expansion allows us to see Tables, Views, Stored Procedures, and Functions, as shown in figure 3.1.

##### Figure 3.1 The database name has been expanded to show Tables, View, Stored Procedures, and Functions.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH03_F01_Iannucci.png)

The triangle next to Tables points right, which means that the view of the contents below Tables has been collapsed. To see the columns in the title table or any other table, we need to click that triangle to expand the list of Tables. Then we'll need to find the title table, click the triangle next to it to expand further, and click the triangle next to Columns to expand it as well. When we complete all those clicks, we see the names of all columns in the title table, as shown in figure 3.2.

##### Figure 3.2 The Navigator panel, where Tables has been expanded to show individual table names and the title table has been expanded to show all columns in that table

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH03_F02_Iannucci.png)

### 3.2.1 Retrieving an individual column

Now that we know the column names, we can start querying the table. Let's begin with a simple query of the TitleName column from the title table. Because we're going to be increasing the length and complexity of our queries, we'll start formatting our queries with the `FROM` clause on a separate line to make it a little more readable:

```sql
SELECT TitleName
FROM title;
```

##### Try it now

Write and execute the preceding query. Consider this query your second one (not that anyone is counting).

Executing the query results in the eight rows shown in figure 3.3. If you happen to see the same eight rows in a different order, don't be alarmed. There is no implicit guarantee of ordering the results of a query.

##### Figure 3.3 The values of the TitleName column in the title table are returned in no particular order.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH03_F03_Iannucci.png)

##### Warning

I'm going to say it again because many SQL users have a misconception: there is no implicit guarantee of the order of results of a SQL query. You shouldn't be surprised when you execute the same query at different times and get the same results in a different order. This can occur due to any number of factors, from modifications in the values of the tables being queried to changes in the settings of the server with regard to your database. Remember that SQL is a declarative language: if your RDBMS isn't explicitly told how to order the results, the rows can appear in random order. That said, if you've peeked ahead in this book, you already know that we're going to discuss how to order results in chapter 4.

Something else to note is that the name of the column in the query is TitleName. You may wonder why it isn't just Name. The main reason is that lots of columns in databases contain data with values for names of things or people, and TitleName is specific to this table. Another reason relates to what I said earlier about reserved words: the word `Name` is one of those reserved words.

##### Try it now

If you executed the previous query, take a moment now to click near `TitleName` and delete the letters in the Title prefix, leaving only `Name`. Did you notice that `Name` now appears in the same color as `SELECT` and `FROM`? That's because in MySQL, the word `Name` is a reserved word.

`Name` isn't a keyword command like `SELECT` or `FROM`, but it's a reserved word relating to a specific action that is done in this RDBMS. For this reason, avoid using the word *Name* for a column name.

##### Tip

Here's a neat trick that can save you some typing in the future: delete "Name" from your query as well, leaving a couple of spaces between `SELECT` and `FROM`. Now move the cursor to the Navigator panel. Click and hold TitleName in the Columns list; then move the cursor to the space between `SELECT` and `FROM`. When you release the click, you should see *TitleName* appear as before without doing any typing. This shortcut is great to use, especially when you're dealing with long column names, and it works for objects such as table names too.

### 3.2.2 Retrieving multiple columns

Until now, we've selected only one column of data, but you'll want to write SQL queries that select multiple columns. Let's think about declaring a statement as we did in chapter 2: "I would like all the TitleNames and Prices of the titles."

We already know how to convert most of this statement into a query, so all we need to do now is consider replacing the word *and* with a comma. Because we have multiple column names, let's change the formatting a bit to make multiple column names more readable. Our query will look like this:

```sql
SELECT
    TitleName,
    Price
FROM title;
```

The comma tells the RDBMS that our query will request another column, just as the word *and* does in the English language. When speaking, we wouldn't end a set of words with *and*. We wouldn't say, "I would like all the TitleNames and Prices and of the titles" because the listener would think, "And what?" They'd know that something else should be included, but it's not clear what that something would be. For the same reason, don't put a comma after the last column in your `SELECT` statement. Doing so will result in a syntax error.

Also note that column order output is completely up to you, the SQL query writer. Just because the columns are in a certain order in the table doesn't mean that they can't be rearranged like this:

```sql
SELECT
    Price,
    TitleName
FROM title;
```

We can even include the same column multiple times if we want, like this:

```sql
SELECT
    TitleName,
    TitleName,
    Price
FROM title;
```

Having two columns with the same name does introduce a bit of confusion, though. Is there a better way to manage multiple columns with the same name? Why, yes, there is!

### 3.2.3 Renaming output columns using aliases

Although your `SELECT` query can't change the names of the columns in the tables you are querying, you can easily change the name of the output column to whatever you want. Let's declare a statement again: "I would like all the TitleNames as BookNames of the titles."

Just as you'd use the word *as* in your declarative statement, you use the word `AS` in your SQL statement to declare the new column name:

```sql
SELECT
    TitleName AS BookName
FROM title;
```

Now your output should show show BookName as the column name instead of TitleName, as in figure 3.4.

##### Figure 3.4 The values of TitleName are now returned with the column header of BookName.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH03_F04_Iannucci.png)

What we've done here is use an *alias*, which is a simple method of renaming the output column from its original column name. We can use column aliases to prevent confusion from similarly titled columns by giving the output columns unique names:

```sql
SELECT
    TitleName AS BookName,
    TitleName AS AlsoBookName,
    Price
FROM title;
```

You don't need to use the word `AS` to use an alias, however. You can put the *alias name* after the query, although this format makes your column aliases a little less obvious:

```sql
SELECT
    TitleName BookName,
    TitleName AlsoBookName,
    Price
FROM title;
```

Column aliases can be wonderful tools for making column names more effective. Just remember to avoid using reserved words as your aliases.

### 3.2.4 Retrieving all columns

We've seen how to select a single column and multiple columns of data from a table. Often, you'll find that you need all the columns of data in a table to be meaningful, and you want to retrieve them all. Perhaps you need to write a detailed report that includes the maximum amount of sales data, or maybe auditors have asked you to supply every bit of customer data. Whatever the case, you have three ways to do this.

The first way is to type out all the column names, separating them with commas. Unless the column names are short and you're very proficient at typing, this approach is the hardest one.

The second way involves much less typing. Select the first column in a table in the Navigator panel, click it, hold down the Shift key, and click the last column. You should see all those columns highlighted, as they are in figure 3.5.

##### Figure 3.5 The column names in the title table, highlighted after selecting the first column name, holding down the Shift key, and clicking the last column name

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH03_F05_Iannucci.png)

Now click and hold any of the highlighted columns, drag the cursor between `SELECT` and `FROM` in the Query panel, and release. You should see all the column names listed in your query, as they are in figure 3.6.

##### Figure 3.6 The columns of the title table after they've been pasted into the Query panel

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH03_F06_Iannucci.png)

##### Try it now

Execute this query, and you'll see the values of all the columns in the table. If the right side of the query is blocked by another panel, you should be able to remove it by choosing View > Panel > Hide Secondary Sidebar.

The third method is probably the most common, as it involves less typing and clicking. You can see all columns in a table by using an asterisk in place of column names, commonly referred to verbally as "select star." This method is also called "select all," although less commonly:

```sql
SELECT
    *
FROM title;
```

This method gives you the same results as the preceding query, and I'm sure you can see why it's so popular. You can easily see the values for all columns with less effort than it takes to type a single column name!

Aside from minimal effort, the other benefit of this method is it allows us to see all the column names in a table quickly without using the Navigator panel. There are two significant reasons to be careful with `SELECT` `*`, however:

* You're selecting all the data, which means that the RDBMS has to read more data and send it across some network to you as output. It may seem that resources are infinite, but in my experience, the use of `SELECT` `*` on very large tables uses so much of the system resources that it causes performance problems for other queries. Be very careful when using this method.
* The second hazard of using `SELECT` `*` is that because it doesn't specify any column order, it should never be used in reusable queries. Suppose that you write a SQL query for a report that expects five output columns from a table. If one or more columns are added to or removed from the table later, your report may no longer work because the number of columns will be different.

##### Warning

For the reasons noted earlier, you should use the `SELECT` `*` method only in ad hoc queries and sparingly even then. As shown in the second method, it's not difficult to click and drag the column names you want to use in a query.

## 3.3 Lab

1.  Another table in the sqlnovel database is named author. What two methods could you use to find the names of columns in this table?

2.  You need to write a query that returns all the first and last names from the author table. How would you write that query?

3.  What do you think would happen if you forgot to put a comma between column names? Do you think this query will work, and if so, what would the output be?

```sql
SELECT
    TitleName
    Price
FROM title;
```

4.  What would happen if you try to use the `SELECT` `*` method with an alias, as in the following query?

```sql
SELECT
    * AS Everything
FROM title;
```

## 3.4 Lab answers

1.  The best method would be to expand the columns folder under the author table in the Navigator panel. If you're using an interface that doesn't allow you to do that, you can use the `SELECT` `*` method in the following query:

```sql
SELECT
    *
FROM author;
```

2.  The answer is

```sql
SELECT
    FirstName,
    LastName
FROM author;
```

3.  This query will execute, but the results won't be what you expect. Because the comma is no longer between `TitleName` and `Price`, the word "Price" is now considered to be a column alias for `TitleName`. Only one column with an aliased name of `Price` will be returned, but the values will be from the TitleName column.

4.  This query won't execute because you can't alias column names with `SELECT` `*`. If you try this query, your first indication of a problem will be the red square with a white X on the line with the alias. This red square means the Workbench application found that your query has incorrect syntax.

