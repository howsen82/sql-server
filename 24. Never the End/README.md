# 24 Never the End

We've arrived at the final chapter of *Learn SQL in a Month of Lunches*. I hope that this book has been useful to you and convinced you that even with little or no programming experience, anyone can learn to write useful SQL queries.

Starting with chapter 1, the goal was for you to be immediately effective in writing SQL queries. With all the concepts and keywords discussed and used, you should feel confident enough to write queries that satisfy a wide range of requests. Now you know different ways to filter, join, and group data, as well as how to modify data and even create objects such as tables and stored procedures. I'm confident that you've learned enough to understand most examples of SQL that someone else wrote.

Still, the end of this book is hardly the end of your exploration of the SQL language. This is truly the beginning because the more you work with SQL, the more new, exciting keywords and objects you'll discover. Where do you go next? Well, here are a few ideas.

## 24.1 More SQL

As you must have noticed, the MySQL Workbench Navigator has a section named Functions that I never addressed. Although you worked with dozens of functions throughout this book, such as `CONCAT` and `COALESCE`, those functions aren't included because they're system functions, and the Functions section is for user-defined functions. That's right—you can create your own functions! As you progress in your experience with SQL, you'll encounter requests that require evaluating values or expressions with a function that you need to create.

Another consideration for future learning is window functions. Although these functions aren't available in every relational database management system (RDBMS), when you're working with an RDBMS that includes them, you can perform powerful calculations such as running totals, rankings, and percentiles for each row. In some ways, these functions operate like a cursor but without the drawbacks of locking, blocking, and excessive resource use.

When you need to construct SQL based on unknown conditions, many RDBMSs offer you the option to use dynamic SQL. As strange as it sounds, *dynamic SQL* allows you to create a string of SQL to be executed later. Although it's somewhat unusual, this technique gives you another level of flexibility in your SQL, which can be useful when a query needs to dynamically change filtering clauses or even the names of tables being queried.

These are a few of the many tools and techniques yet to be discovered on your SQL journey. Where should you go next?

## 24.2 Other SQL resources

The best way to increase your skill level in any language, whether it's spoken to a computer or to another person, is to practice. I added labs in nearly every chapter to get you to practice thinking about SQL and using it to solve problems. You can continue to practice by using the sqlnovel database to write SQL to do things like insert new rows into the orderheader and orderitem tables and retrieve data for sales by category. The practice possibilities are limited only by your imagination.

Then again, your immediate need may be to work with an RDBMS other than MySQL, in which case you can install a tool that allows you to work with that RDBMS and find a sample database to use for practicing SQL queries. Free sample databases are available for every RDBMS, so use your favorite search engine to find them, and use these databases to write your own practice queries. The more you practice writing SQL, the easier it will be to respond to any request effectively.

If you found this book helpful, take a look at other RDBMS-specific books from Manning that can help you improve your SQL skills, such as *100 SQL Server Mistakes and How to Avoid Them*, by Peter Carter (<https://www.manning.com/books/100-sql-server-mistakes-and-how-to-avoid-them>), and *PostgreSQL Mistakes and How to Avoid Them*, by Jimmy Angelakos (<https://www.manning.com/books/postgresql-mistakes-and-how-to-avoid-them>). As I've noted throughout this book, every RDBMS is slightly different in terms of SQL syntax, and an RDBMS-specific book can increase your depth of knowledge in ways that can benefit your career. Although it's good to have broad knowledge of the SQL language in the ways I've discussed throughout this book, obtaining most of your experience in a particular RDBMS could allow you to showcase yourself as an expert in that particular flavor of SQL.

Perhaps you'll discover that you want to move beyond writing queries that retrieve data and learn about creating databases. If so, consider books such as *Understanding Databases*, by David Clinton (<https://www.manning.com/books/understanding-databases>) and *Grokking Relational Database Design*, by Qiang Hao and Michail Tsikerdekis (<https://www.manning.com/books/grokking-relational-database-design>), that cover ways to design databases that perform well and scale with the massive amounts of data that modern databases contain. This book didn't discuss database design in depth, but understanding the capabilities and limitations of a database is also an important skill.

Above all, no matter what you choose to do next, be curious, and don't stop learning.

## 24.3 Farewell

It's been my pleasure to help you begin what I hope is a long-lasting journey using the SQL language. Whatever the future holds for you, I congratulate you on all the work you've done so far, and I wish you the best in whatever comes next!

