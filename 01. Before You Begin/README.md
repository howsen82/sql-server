# 1 Before You Begin

Nearly every act of our lives generates data. Every purchase we make, every mile we travel, and every internet link we click adds to a colossal amount of ever-growing data, which for many organizations has become their most valued asset. This data is often stored in a relational database, which keeps the data secure, scalable, and available to be constantly read and modified by innumerable users.

But how exactly can these users work with the data in a relational database? More important, how can you read and write data that is critical to your organization? The answer, and the subject of this book, is *Structured Query Language*, more commonly known as *SQL*.

## 1.1 Why SQL matters

Now, you may be wondering whether SQL is important enough for you to invest an entire month of lunches in it. Be assured that learning this language is undoubtedly one of the best skills anyone who uses data can acquire. Even though much of the data of our modern lives is stored in relational databases from different brands, such as Oracle, Microsoft, and IBM, nearly all use SQL to work with data and have been using it for quite some time.

Although many application languages have a lifespan of only a few years, SQL has been the standard language for querying relational databases for decades and should continue to be so for the foreseeable future. This means the skills you'll learn and develop by reading this book and practicing the recommended exercises can potentially benefit you for your entire career.

Perhaps most important, SQL is very easy to learn because it was designed to be written like the English language. If you are familiar with English, the commands and syntax used in SQL will seem intuitive. If you need to work with your data to find the first and last names of customers in Canada, for example, you might use a SQL statement like this:

```sql
SELECT FirstName, LastName FROM Customers WHERE Country = 'Canada';
```

See how easy that is? Although you may not understand every bit of that statement, I assure you that within the first few chapters of this book, you will be able to write queries like this one.

## 1.2 Is this book for you?

There is no shortage of books, videos, courses, or websites that offer to teach you SQL, many of which are designed for an audience with software development experience. They frequently start with the history of a language, move on to a discussion of its many technical concepts, and follow with chapters grouped by showing what various commands do. Though nothing is inherently wrong with that approach, it ignores the many nontechnical folks who need to learn SQL—folks like me.

Despite using SQL for more than two decades now, I didn't begin my career in software development. My first experience with databases was in a position called data administrator, where I was responsible for importing data from various sources into a relational database. I needed to read that data to validate the success of the import process, and the only way to do that was by learning and using SQL.

Even though I had limited programming experience, I quickly grasped how to use SQL. If you understand how to write in English, I'm confident that you can do the same. As you will see throughout this book, most SQL commands and keywords are exactly what you would expect them to be in English.

### 1.2.1 The many uses for SQL

Data isn't just for the IT department, of course. If you are a business analyst, for example, you can use SQL to quickly retrieve and analyze data about operational trends to make smarter business decisions. If you are a marketing professional, you can use SQL to uncover actionable insights about recent ad campaigns that can help you grow your business. If you work in finance, you can use SQL to retrieve vital data that can help your company meet compliance requirements.

All this data is the lifeblood of any modern organization, and success depends on having members of nearly every department possess the skills to use relational data to make critical business decisions. This book is designed to help people like you learn SQL to build those skills. If your technical experience is limited to working with spreadsheets, you're at a great starting point.

Then again, if you are a software developer, database administrator, or data scientist, this book doesn't exclude you; it just takes a different approach to learning. Whereas most other SQL books begin with terminology and concepts, this book gets you using SQL quickly to solve practical problems while briefly sharing concepts and defining terminology along the way.

Conversely, this book isn't designed simply to teach you a bunch of SQL commands. Instead, it's designed to progressively show you how to apply components of the SQL language to do your job, regardless of your level of computer programming experience.

### 1.2.2 The many flavors of SQL

Although we will be using a MySQL database to learn about SQL, nearly all the SQL concepts and techniques will work with any relational database. This means that what you learn will apply to any of the following database management systems:

* IBM DB2
* MariaDB
* Microsoft SQL Server
* MySQL
* Oracle
* PostgreSQL

When you've developed a solid foundation for using SQL, you can easily work with data in any of these systems. Be aware, however, that there will be occasional exceptions for individual systems. These exceptions will be noted throughout this book so you can be proficient in whatever system you use to work with data.

### 1.2.3 A word about AI and SQL

With the advent of generative artificial intelligence (AI), you may be wondering why you should learn to use SQL instead of using a tool like ChatGPT to write any SQL you might need. Though AI seems like a handy way to avoid investing in learning SQL, you still need to have a good grasp of SQL to understand whether the code that any such tool provides will give you correct results. Moreover, to get a SQL statement from a generative AI source, you have to provide details about your database, which your organization may expressly prohibit.

That said, generative AI tools can be useful when you have a good understanding of a language such as SQL. As you become proficient and understand how to write SQL that does exactly what you intend, you can use these tools to quickly review your code for performance problems or explain what a query appears to be doing. You still need to possess SQL knowledge to interpret the recommendations or explanations, and this book can help you attain that knowledge.

## 1.3 How to use this book

The idea of this book is that you will read one chapter each day. You don't have to read during lunch, but most chapters should take about 40 minutes to read, leaving you about 20 minutes to practice what you've learned while you finish eating.

### 1.3.1 The main chapters

Chapters 1 and 2 help you get up to speed quickly, making you familiar not only with the idea of a table and how to think about querying it but also with the tools we will be using throughout this book. In some ways, they're the most important chapters of the book.

Chapters 3 through 22 represent the primary content, so you can expect to complete them in about a month—even a short month like February. Not every chapter will require a full hour, but it's important to follow the order because each chapter builds on the skills and commands demonstrated in previous chapters. Also, though you are certainly free to read multiple chapters per day, I recommend focusing on a single chapter daily and spending ample time practicing what you learned. Doing this will give your brain time to focus on a handful of concepts and examples, which should prove to be optimal in solidifying your knowledge quickly. As the great basketball coach John Wooden said, "Be quick, but don't hurry."

### 1.3.2 Hands-on labs

Nearly all chapters include a short lab exercise that helps you apply the concepts and commands you've learned. Don't think of these exercises as quizzes but as opportunities to apply and reinforce your new SQL skills. Though the answers to these labs appear at the end of each chapter, I can't stress enough how vital working through these labs will be in retaining your new knowledge.

### 1.3.3 Further exploration

Because this book is designed for those who are just starting to use SQL, it only scratches the surface of the ways you can use and manipulate relational data. For this reason, some chapters end with suggestions for further exploration of ways to use the concepts and commands. If you have the time and inclination, take a look at these resources to expand your ever-growing SQL skill set.

## 1.4 Setting up your lab environment

Your time is valuable, so let's get started with setting up your lab environment. This task won't be resource-intensive, and you can likely set it up on your own computer in a few minutes. We'll install only two pieces of free software and then execute some ready-made SQL scripts to give us some data to use.

### 1.4.1 Installing MySQL and MySQL Workbench

The first step is downloading MySQL and installing it on the computer of your choice. MySQL is not only freely available but also one of the most popular relational database applications in the world.

We'll also install MySQL Workbench, which is the tool we'll use to execute all the queries contained in this book. It also uses very few resources, so you shouldn't worry about installing it on a laptop.

The steps for downloading and installing both of these applications are available at my GitHub repository, located at <https://mng.bz/PNl8>. Because the MySQL software is frequently updated, the version numbers you see may be later than the ones shown in the documentation. Don't worry about that; nothing we do should be affected by newer versions.

### 1.4.2 Executing the lab scripts

Throughout this book, we'll rely on a single set of data for our queries. The data is based on a set of orders from a hypothetical publisher of SQL-based novels, using a database named sqlnovel for all our queries. We'll discuss this data more throughout the book, but for now, let's create the database and populate the sample data by executing a prepared SQL script.

The steps for setting up our sqlnovel database are also located at <https://mng.bz/PNl8>, and they're even simpler than the process for installing MySQL and MySQL Workbench. Although you are likely to simply execute the script, near the end of the book, we will review parts of the script to examine what it does. By that point, you should be able to create your own sets of data!

## 1.5 Online resources

Throughout the book, I'll give you examples and exercises to try. I encourage you to type all scripts on your own and even to write SQL in a different style from the one I present in this book if you prefer. When you type the SQL, you may encounter an error that you don't understand. For this reason, the online resources also contain every SQL script presented in this book. Please try to use them only for troubleshooting because typing the SQL yourself will help you learn faster than simply copying scripts.

## 1.6 Being immediately effective with SQL

As with every other book in the Month of Lunches series, the primary goal of this book is to make you immediately effective. Nearly every chapter that follows presents a particular part of the SQL language and discusses it briefly, though most of any given chapter focuses on how to apply what you've learned using real-world scenarios. Furthermore, at the end of every chapter, you get hands-on practice by completing exercises in a lab environment.

As stated earlier, if you are looking for a deep dive into relational database theory and history, many other books can guide you down that path. Although many parts of this book discuss details and nuances, every chapter is driven by the goal of making you immediately effective at accomplishing real tasks.

OK, that's enough about this book. Let's start using SQL!

