# 2 Your First SQL Query

Chapter 1 ended with a word about being immediately effective, and so now we're going to do just that. We're going to start looking at some data as it might be stored in a relational database, and we're going to examine the way that data is structured. Doing this will help you better understand some terms to describe the data, which we will use throughout the book. Don't worry, though: you'll see just a handful of terms, and they are all words you have seen and used in conversation. I'm just defining them in the context of data stored in relational databases.

Also, you'll get started with your first query. In case you didn't know, a *query* refers to executing some SQL to retrieve data. As you progress through this book, you'll execute quite a few queries to level up your SQL skills. If you haven't already completed the installations of MySQL and the MySQL Workbench (see chapter 1) and executed the `Create_SQLNovel_database.sql` script to create our sample database, please do those things now so that you'll be ready to query data. Before you begin querying, though, let's look at some data.

## 2.1 You know tables if you already know spreadsheets

Although it's not a prerequisite for learning SQL in this book, it will be helpful if you have experience working with Microsoft Excel or some other spreadsheet program. You may not realize it, but spreadsheets are structured similarly to the most fundamental objects in any database. We'll also introduce a few terms in this section to help make sense of the way data is stored in a relational database—more accurately known as a relational database management system (RDBMS).

Now, we don't just gather data and dump it into an RDBMS; rather, we organize and store it in objects based on the nature of the data. These objects are known as *tables.* We typically organize data in tables relating to elements, such as orders, customers, or payments. Tables are the building blocks of any RDBMS and are structured quite a bit like spreadsheets, so looking at a spreadsheet will help you understand the associated terms in this chapter and throughout the book.

If you don't know the basic terms used to describe a spreadsheet, take a look at the typical spreadsheet in figure 2.1, which contains information about some extraordinary fictional books. Consider this information a set of data, commonly known as a *data set*. Seems easy enough, right? The data set is stored in a spreadsheet, but had this data been stored in a table, it would have essentially the same structure.

##### Figure 2.1 A spreadsheet with five columns (A through E) of values for several fictional books. The spreadsheet is organized similarly to a table in a database.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH02_F01_Iannucci.png)

I've said that I don't want to overload you with jargon, but you need to understand three simple but critical terms related to tables before you start using SQL to read and manipulate any data contained in tables. As I noted at the beginning of the chapter, you've likely heard these words before:

* Column
* Row
* Value

A table, at its most basic level, is a construct of one or more *columns* of data. Columns run vertically, like columns in architecture. In figure 2.2, we see columns for Title, Price, Advance, Royalty, and Publication Date, with Title highlighted. You may see or hear the term *field* used to refer to a column, but *field* isn't a term in the SQL language.

Another term we need to consider is *row*, which refers to a horizontal collection of data in the table. Each row represents a single item of whatever the element of the table is, which in this case is the title of a book. In figure 2.3, we can see that each row has the same structure and follows the same order of columns—a requirement because each row must include a representation of all columns in the table.

##### Figure 2.2 The spreadsheet of book titles, with the Title column highlighted to show the vertical nature of columns

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH02_F02_Iannucci.png)

These rows are also enumerated in the left sidebar, but in any given table, the designer may not include explicit identifiers for each row. It's worth noting that the terms *row* and *record* are often used interchangeably because certain applications refer to rows as *records,* but for tables in most RDBMSes, the correct term is *row.*

##### Figure 2.3 The spreadsheet of books, with the first horizontal collection of data highlighted to show the horizontal nature of rows

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH02_F03_Iannucci.png)

The last term, at least for now, is *value*, which represents the distinct pieces of information described by the columns of the data set. Every row contains one value for each column. In figure 2.4, the value of Title in the last row of our data set is The Sum Also Rises, and the value of Price in that row is $7.95. It's worth noting that even though all columns are required for all rows, the values for the columns can be empty, such as the Advance value for the row with the Title value The Great GroupBy.

##### Figure 2.4 The value of Price in the row with the title The Sum Also Rises highlighted to indicate a value. This value is just one of many.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH02_F04_Iannucci.png)

OK, that's enough terminology about tables for now. Let's start talking about how to use this information to query a table.

## 2.2 Learning SQL is like taking an English class

A common question many people have is how to pronounce *SQL;* some folks say "ess-cue-ell," whereas others say "sequel." Considering that the earliest version of SQL was called Structured English Query Language and was abbreviated as SEQUEL, you can see how the latter pronunciation became commonplace. For what it's worth, there was already a trademark on SEQUEL, so the creators dropped the word *English* from the name and shortened the abbreviation to SQL.

This brings us to another reason for the popularity of SQL: unlike many other programming languages, it's designed to resemble the English language. You see, SQL is a *declarative* language, meaning that you specify what data you want and not how you want to get it, which is something that the RDBMS you are using will figure out.

We can take this concept of SQL being a declarative language a step further, using simple verbal declarations to say what we want to do with the data. This may seem unusual, but you'll soon see that many basic SQL statements are similar to a verbal declaration for a simple request. Let's walk through an example. Suppose that you have a table of vegetables named vegetables, like the one shown in figure 2.5, and you want to know the names of all the vegetables. If you want to declare this request verbally, you might say something like this: "I would like all the names of the vegetables."

##### Figure 2.5 A vegetables table with two columns and five rows. We're going to learn how to create a query that shows the names of all the vegetables.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH02_F05_Iannucci.png)

SQL isn't intuitive enough for that to work, but it isn't too far off. To accomplish this hypothetical query, you need to include in your declaration the two most basic keywords used in SQL. A *keyword* is any word in the SQL language that helps you do . . . well, anything. The first keyword to learn is `SELECT`, which, when it comes to databases, will be your new best friend. Believe me—you and `SELECT` will work together a lot. Why? Simply put, `SELECT` is the keyword used to define what you want to see and how you want to see it: "I would like to SELECT all the names of the vegetables."

All right, we're on our way to forming an actual SQL query, but we need to add something else: the `FROM` keyword. The `FROM` keyword specifies which data set we want to look at, which in this case is the vegetables table: "I would like to SELECT all the names FROM the vegetables table."

That's better, but when we specify a data set using `FROM`, we don't explicitly say that it's a table. Even though tables are one of several kinds of data sets you can query, your RDBMS can determine the kind of data set based on the name of the data set: "I would like to SELECT all the names FROM vegetables."

We're getting closer. Now let's consider "SELECT all the names" for a moment. If we want to select all the names, we're in good shape because that is the default for this type of query. Nothing here specifies that we want any particular names, so we don't need to state that we want them all: "I would like to SELECT names FROM vegetables."

This part is tricky because we'll need to look at the table in figure 2.5. We can see that the name of the column with the data we want is called Name, not Names. As you query more data, you'll probably find that a column name is hardly ever plural because the value in each column rarely has more than one value for each row. Let's adjust our verbal declaration a bit: "I would like to SELECT Name FROM vegetables."

We're almost there. The last modification is to remove the "I would like to . . ." text because we start queries with a keyword, which in this case is `SELECT`. Also, that part is kind of wordy, don't you think?

One more thing: we need to add a semicolon to the end of the declaration. The semicolon tells the RDBMS that this is the end of what we're declaring and that anything else after it is another query:

```sql
SELECT Name FROM vegetables;
```

There you go! This is the correct way to query the names of all the vegetables. Next, let's go from designing a query for hypothetical data to writing and executing a query on actual data.

## 2.3 Writing your first SQL query

For the first bit of actual SQL you're going to write and execute, simply seek the outcome of your first query. As noted previously, we can start by declaring a sentence in English that defines what we want: "I would like the outcome of my first query."

Fortunately for us, the data to be queried already exists in a table named MyFirstQuery, and the values are in a column named Outcome. Convenient, right? Using what we learned about SQL syntax in section 2.2, we can easily craft a simple query to accomplish our goal:

```sql
SELECT Outcome from MyFirstQuery;
```

As you can see, the query ends with a semicolon. To add a little more to what I said earlier, in SQL, a semicolon is used as a *statement* *terminator*. We don't need to go deep into this subject; just know that a semicolon effectively means we're done with this statement and anything that comes after it is another SQL statement. Doing this prevents confusion for the database engine (especially when we get into more complicated statements later), so we'll use semicolons as statement terminators in all our SQL queries.

You may wonder what the difference is between a *statement* and a *query* and whether these terms are interchangeable. Well, statements and queries aren't the same things, but they're related. Think of a query as a special kind of statement for retrieving data. As you advance your SQL skills, you'll find yourself using statements beyond queries. For now, though, queries are the only SQL statements you'll use.

##### NOTE

Depending on which RDBMS you're using, a semicolon may not be required as a statement terminator. Although the RDBMS you use may not require it, it's good to develop the habit of ending all SQL statements with a semicolon—even statements as simple as your first SQL statement.

Before we go any further into the weeds on statements, let's get back to our query. Now that we have our query, the next thing we need to do is open MySQL Workbench so that we can execute the query. *Executing* is like clicking Send in your instructions to the RDBMS, which will figure out the best way to complete your query and then return the results to you. Those results are displayed in a different window in MySQL Workbench.

##### Try it now

Open MySQL Workbench, and click to open the Month of Lunches connection we created in chapter 1. Alternatively, right-click Month of Lunches, and choose Open Connection from the pop-up menu. You should see something like figure 2.6, with Query 1 highlighted. That represents the top of the Query panel, including the number 1 in the panel. That number indicates the first line of any query we enter here.

##### Figure 2.6 The MySQL Workbench open with our Month of Lunches connection, with administration information shown in the Navigator panel. We enter queries like the one we just wrote in the Query panel.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH02_F06_Iannucci.png)

I'd like to point out a few things in MySQL Workbench. The first item is the tab at the top that says *Month of Lunches*. This tab tells us the context of our connection, which we set up in chapter 1 for the lunch user. We're not going to change that context for any exercise right now, but if you find yourself working with MySQL beyond the scope of this book, you'll always want to pay attention to the connection you're using when querying data.

The second item I want to focus on is the left side. You can see the Administration panel, but the tab next to that one, named Schemas, is more important to us. Click the word *Schemas*, and notice the sqlnovel database here. This database was created in the scripts we executed in chapter 1, and we want to set it as the default database for all our queries. To do this, right-click sqlnovel and choose Set as Default Schema from the pop-up menu. Your MySQL Workbench screen should look like the one in figure 2.7.

##### Figure 2.7 The Month of Lunches connection again, now with the Schemas information in the Navigator panel and the sqlnovel database shown in bold text, indicating that it's the default database

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH02_F07_Iannucci.png)

As we progress through the book, I'll take time to point out more information that is contained in MySQL Workbench. For now, though, verifying the connection and the database we'll be using is enough. Let's get back to the query:

```sql
SELECT Outcome FROM MyFirstQuery;
```

##### Try it now

Move the cursor to the Query panel, and click to the right of the 1 and the blue dot. Enter your first SQL query here by typing the query that precedes this sidebar. It should look like figure 2.8.

##### Figure 2.8 The query entered in the Query panel, ready to execute

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH02_F08_Iannucci.png)

I know your anticipation is building, so let me assure you we're almost done. We can execute the query in one of several ways. First, we can select Execute All or Execute Current Statement from the Query menu at the top. For this single query, both commands do the same thing because we have only one statement in the Query panel.

As you may have noticed, the Query menu has a few hotkeys we could use to execute the query. Pressing Ctrl+Enter on your keyboard will execute the part we selected, and pressing Ctrl+Shift+Enter will execute the contents of the entire panel. Again, because we have only one line, these shortcuts effectively do the same thing for this particular query.

Finally, you may notice some buttons directly above the first line of the Query panel. Several of them look like lightning bolts, but let's focus on the first one on the left. That plain lightning bolt, highlighted in figure 2.9, does the same thing as pressing Ctrl+Enter: it executes the selected part of the Query panel.

##### Figure 2.9 The highlighted plain lightning bolt in the Query panel. Clicking the plain lightning bolt executes the selected part of the Query panel, which does the same thing as pressing Ctrl+Enter on your keyboard.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH02_F09_Iannucci.png)

##### Try it now

Make sure that you place the cursor at the end of the SQL statement before you choose your method of execution. Go ahead—do it!

After executing the query, you should see information in two new panels below the query. Your Workbench screen should look like figure 2.10.

Below the Query panel is the Result panel, and as you see, the result of the first query is "Hello, World!" If you aren't a computer programmer, you may not know that one tradition in learning a computing language is to first learn how to get "Hello, world!" as output in that language. SQL may not be like most programming languages, but we're still going to be respectful of traditions.

##### Figure 2.10 The result of query execution. In the Result panel, we see the result is "Hello, World!" In the Output panel, a circle with a check mark (which appears green onscreen) indicates the query executed successfully, and other information shows the time it was executed, what the query was, how many rows it returned, and the duration of query execution.

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH02_F10_Iannucci.png)

Look at what's immediately above "Hello, World!," though. It's the word *Outcome,* which indicates the column that we selected to query. This result, as small as it is, is still considered a data set. It's one column and one row, but it's not a table. We queried a table named MyFirstQuery, but the result of the query is a separate data set all by itself.

One final thing to notice about our execution is the Output panel below the Result panel. This panel provides several bits of information, such as the time when our query was executed, what the query was, how many rows were returned, and how long it took to execute the query. Most important is the circle with a check mark (green onscreen), which indicates that the query executed successfully. If our query hadn't executed successfully, we would have seen a circle with an X (red onscreen) to indicate an error. Ideally, we won't see many circles with an X as we work through this book.

## 2.4 Key terms and keywords

In this chapter, we've covered a handful of simple terms and a few keywords to get you on your way to querying data. Because these concepts and commands are the building blocks of SQL, let's take a moment to review them:

* *Data set*—A *data set* is a logical grouping of data that can be contained in a database, a spreadsheet, or any number of other places. As far as we're concerned, we're going to use SQL to query data in an RDBMS.
* *Table*—A *table* is a logical construct of columns that contains a data set. It's the most fundamental way to store data sets in an RDBMS.
* *Column*—A *column* is the vertical grouping of attributes for every row in a table.
* *Row*—A *row* is the representation of related data in a table.
* *Value*—A *value* represents the actual data for a column in a row.
* *Statement*—A *statement* is a way to declare to the RDBMS that we want to do something.
* *Query*—A *query* is a special kind of statement used to retrieve data.
* `SELECT`—The `SELECT` keyword starts queries. The next few chapters of this book delve deeper into ways to use the `SELECT` keyword.
* `FROM`—The `FROM` keyword identifies the data set that we want to query.
* *Semicolon*—Don't forget to end your queries with a *semicolon!*

## 2.5 Lab

Throughout the rest of the book, I will end each chapter with one or more lab exercises to put into practice what we have learned. These exercises are intended to emulate the practical use of SQL, simulating scenarios you would encounter using other people's data. Because this lab is the first, though, we'll start off easy with a few simple mental exercises to get a little more familiar with tables and their structure.

Considering what we've learned about using the second `SELECT` statement, or even considering the example in figure 2.1 to be a table, take a moment to ponder the following conceptual questions:

1. Imagine that spreadsheet in figure 2.1 is a table with several columns. Also, note that the table used in our second `SELECT` statement has at least one column. Is it possible for a table to have no columns?
2. Now imagine that either of these data sets has no rows that contain data. Is it possible for a table to have zero rows?
3. In figure 2.1, one cell in the Advance column does not have a value. If this figure were a table, do you think it would be required to have a value?
4. Assuming that we have a vegetable table in our sqlnovel database with a column named Name, what do you think would happen if we combined the two queries we discussed in this chapter and executed them at the same time? Think about the result of a query such as the following:

```sql
SELECT Name FROM vegetable;
SELECT Outcome FROM MyFirstQuery;
```

Do you think that this SQL would execute successfully?

## 2.6 Lab answers

1. No, it isn't possible to have a table with no columns. This is why I mentioned earlier that you can think of a table as a collection of columns. There must always be at least one column; otherwise, there's nowhere to put the values of data.
2. Yes, you can have tables without rows. Every time a table is created, it starts with no rows.
3. This question is a bit tricky because the answer depends on the way the table was set up. We can have rows of valid data that don't have values for particular columns. Think of a column for Middle Name in a table of people. Not everyone has a middle name, so we have to be able to accommodate that lack of data for rows with people who have no middle name. That said, designers can put restrictions on tables to require all rows to have a value for particular columns, such as a column that captures a last name for all rows in that same table of people.
4. These queries will execute, resulting in two separate data sets being returned. This is the very reason why we put those semicolons at the end of our queries: to tell the RDBMS that we have to separate the queries we're executing.

All right, let's move on to chapter 3 and learn more fun ways to query using the `SELECT` keyword!

