# 7 Filtering with Wildcards and Null Values

The preceding chapters are filled with different ways to filter the data returned by your queries using numerous comparison operators. We've worked with many methods for filtering on one or more values of equality or inequality using known values or ranges of values. Let's take one more chapter to examine some interesting ways to search for less specific data.

We'll look at how to filter data when we don't know the exact values to be searched. Instead of searching for specific values, we'll search for *patterns of values*. This approach can be incredibly useful when we want to look for a list of products that have specific text like *tomato* or *cable* in the name, or when we want a list of all customers whose last name starts with the letter *A.*

We'll also look at the trickiest value to search on: null. Null values are commonly misunderstood, and as such, they often lead to incorrect query results. We'll examine what a null value is (and isn't) and how to query for null values.

## 7.1 Filtering with wildcards

In chapter 6, you learned how to search ranges of numeric or date values. Even though you may not know all the specific values you want from a range, you know how to query the correct results using operators such as `>`, `<`, and `BETWEEN`.

Interestingly, we can use those same operators to search for string values. If we want to find all the first and last names of authors with a last name that starts with *S*, for example, we could write SQL using `>=` and `<` to get the result shown in figure 7.1:

```sql
SELECT
    FirstName,
    LastName
FROM author
WHERE LastName >= 'S'
    AND LastName < 'T';
```

##### Figure 7.1 The results of searching the author table for a range of last names that start with *S*

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH07_F01_Iannucci.png)

##### Note

From here on out, we won't sort results unless sorting is necessary. As I stated in chapter 4, sorting data with `ORDER` `BY` increases the work required to process any query, so avoid doing that if possible. Just remember that without `ORDER` `BY`, it's always possible to get the same results in a different order for any given query.

This method of searching for string values in a range works most of the time, but not always. As briefly noted in chapter 6, depending on the collation settings, the character case (upper or lower), and characters used (such as letters with tildes or umlauts), you may not get consistent results using this method to filter on string values.

Also, it just looks weird to write a query this way, and we certainly wouldn't verbally declare how we want to filter results this way. We want a list of last names that start with *S*, not a list of names between *S* and *T.* In this case, a wildcard makes more sense.

A *wildcard* is a special character that can be substituted for any number of characters in a string. Using a wildcard allows us to search for specific patterns of values instead of being restricted to a range, as in the preceding query.

### 7.1.1 Filtering with the percent sign

The first wildcard we'll use is `%`, the percent sign. When used as a wildcard, `%` matches any string, including an empty string with no characters. Here's how we'd use it to find the names of authors with a last name that starts with *S*:

```sql
SELECT
    FirstName,
    LastName
FROM author
WHERE LastName LIKE 'S%';
```

Notice that we're using a new operator, `LIKE`. `LIKE` is the operator we'll always use when searching with a wildcard because in the SQL language, it indicates that we're searching for a pattern, not for precise conditional values. If we tried to use some other conditional operator (such as `=` or `>`) in this query instead of `LIKE`, we'd get no results.

##### Try it now

Execute the preceding query; then try using the `=` operator instead of `LIKE`.

Even though we know how the query ends, let's take a moment to compare it with how we might verbally declare this query: "I would like the first name and last name from the author table where the last names start with *S*."

I can assure you that SQL has no STARTS WITH operator, and although it might be useful for our query, it wouldn't be very flexible. We'd also need hypothetical operators for other queries, such as ENDS WITH and maybe even HAS IN THE MIDDLE. These operators would be excessively wordy, if not a bit ridiculous.

In SQL, the `%` operator is not only shorter but also has the same functionality as all those other hypothetical operators. The easiest way to remember how to use it is to think of the `%` wildcard as the word *something.* The *something* pattern we're looking for could be 0 characters or 100. Here's a verbal way to say what we're doing with the query: "I would like the first name and last name from the author table where the last names are like *S* and then *something*."

This statement is fairly close to what our new query looks like, and the results will be the same as those shown in figure 7.1. As I just mentioned, the `%` wildcard can be used anywhere in a string of characters, which means that if we want to search for all last names that end in *N*, we could verbally declare a query like this: "I would like the first name and last name from the author table where the last names are like *something* and then *N*."

As you can imagine, our query would be very similar to the verbal declaration. Here it is, with the results shown in figure 7.2:

```sql
SELECT
    FirstName,
    LastName
FROM author
WHERE LastName LIKE '%N';
```

##### Figure 7.2 Authors who have a last name that ends with *N*

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH07_F02_Iannucci.png)

If we wanted to refine this search to include only the last names that not only end with *N* but also start with *M*, like the results shown in figure 7.3, we can certainly do that as well:

```sql
SELECT
    FirstName,
    LastName
FROM author
WHERE LastName LIKE 'M%N';
```

##### Figure 7.3 Authors who have a last name that starts with *M* and ends with *N*

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH07_F03_Iannucci.png)

We can also use the `%` operator both before and after a character or string of characters to find a pattern in the middle of our data. Here's an example of searching for authors with the string "de" anywhere in their last name, with the results shown in figure 7.4:

```sql
SELECT
    FirstName,
    LastName
FROM author
WHERE LastName LIKE '%DE%';
```

##### Figure 7.4 Authors who have a last name that contains "DE"

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH07_F04_Iannucci.png)

This technique can be useful for searching a column of comments or other freely entered text. If you want to find any comments that include the word *good*, for example, you'd search for column values `LIKE` `'%good%'`. As I noted in chapter 3, most relational database management systems (RDBMSes) aren't case-sensitive, so you should be able to return values including "Good" and "GOOD." Then again, you might also get string values like "not very good" and "goodbye" in your results because they also match the pattern.

##### Warning

Although the `LIKE` operator isn't case-sensitive in the default collations of MySQL, Microsoft SQL Server, and SQLite databases, it can be case-sensitive in the default collations of PostgreSQL and Oracle databases.

As helpful as the `%` wildcard can be for finding patterns of characters, it lacks precision. If we want to search values at a particular position, we can use a different wildcard.

### 7.1.2 Filtering with an underscore

Whereas the `%` wildcard matches any string of characters (including zero characters), the `_` wildcard, an *underscore*, looks only for any single character. What's more, we can combine `_` with `%` in our search patterns if necessary.

##### Warning

The `_` wildcard is not supported in DB2.

If we want to find the first and last names of any author with a first name that starts with *R* and has *b* as the third letter, as shown in figure 7.5, we can use _ and `%` to find them:

```sql
SELECT
    FirstName,
    LastName
FROM author
WHERE FirstName LIKE 'R_b%';
```

##### Figure 7.5 Authors who have a first name that starts with *R* and has *b* as the third letter

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH07_F05_Iannucci.png)

Although the _ wildcard is generally used less often than the `%` wildcard, you can still face scenarios of searching for patterns at one specific position, such as if you need to find items with a color value of gray, which could be spelled *gray* or *grey*. To find all matching values, you could search `WHERE` `color` `LIKE` `'gr`_`y'`.

You can also search for values or locations that have a difference in the first few characters by using the `_` wildcard. Here's an example of finding any author with a first name that has *u* as the third character, with the results shown in figure 7.6:

```sql
SELECT
    FirstName,
    LastName
FROM author
WHERE FirstName LIKE '__u%'
ORDER BY FirstName ASC;
```

##### Figure 7.6 The results of authors who have a first name that has *u* as the third letter

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH07_F06_Iannucci.png)

Wildcards other than `%` and `_` are supported by each RDBMS, but because they vary, I won't discuss them in this book. That said, if you're using a particular RDBMS, I encourage you to look into what other wildcards it may support to further enhance your ability to search for patterns of values.

Now let's move on to . . . well, nothing.

## 7.2 Filtering with null values

Sometimes, database designers create columns that require values for every column, but at other times, a column may allow for the absence of data. If a row does not have any data for such a column, the value shows NULL for that column.

As I noted at the beginning of this chapter, null values are some of the most misunderstood concepts in databases. Put simply, null values are literally nothing: they represent the absence of data. This concept seems simple, but because null values aren't values like 30 or Arizona or 2012-05-12, we need to consider them differently from other values when querying data.

Let's look at an example by reviewing all the columns in the author table. Executing the following query returns all the columns for all 11 rows in the table. One of the first things you may notice is the MiddleName column, which has quite a few values that say NULL. Not everyone has a middle name, so the absence of a middle name for any author is represented by NULL. MySQL Workbench tries to bring this fact to your attention by making NULL look different from other values we've seen so far, in that it is shown with white text in a smaller font and a dark background (figure 7.7).

```sql
SELECT *
FROM author;
```

##### Figure 7.7 The results of all columns in the author table, including null values in the MiddleName column

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH07_F07_Iannucci.png)

### 7.2.1 How not to search for null values

As I stated in the preceding section, a null value represents the absence of a value, which makes any column containing a null value tricky to query. To avoid some common pitfalls, let's first talk about how *not* to query for null values. If we want to find the rows in the author table that contain null values for MiddleName, none of the next three examples will work:

```sql
/* This doesn't work because null values are not blank strings. */
SELECT *
FROM author
WHERE MiddleName = '';
```

This query won't return null values because the query is searching for a character string with a length of 0, also known as an *empty* string. I know that's confusing: when I mention "absence of a value" and "empty string," it seems that I'm saying the same thing in different ways. But an empty string is different from a null value because an empty string is still a string. By that, I mean that underneath the covers, your RDBMS is still using bytes to indicate a value for the empty string, so it can be considered for queries that are filtering with many comparison operators, including a wildcard search. Null values use no bytes and are not considered for filtering with comparison operators or wildcards. Here's another common but incorrect way to search for null values:

```sql
/* This doesn't work because null values are not the word null. */
SELECT *
FROM author
WHERE MiddleName = 'NULL';
```

This query doesn't work because the search condition is for the word "NULL," not a null value. Also, it won't return any rows unless your data is populated with a string of the four characters that make the word "NULL." That may seem unlikely, but sometimes database developers don't understand how to work with null values, so they use the word "NULL" to represent null values. Because the word "NULL" is a string of characters, this can create all sorts of headaches for your queries. Please don't ever do this.

Here's one last incorrect way to search for null values:

```sql
/* This doesn't work because no value ever equals null. */
SELECT *
FROM author
WHERE MiddleName = NULL;
```

It seems like this query should work, but `=` is looking for equality, and you can't have equality matches of nothing. At the most basic level, all the comparison operators, including `=`, are evaluating for search conditions that are either true or not true. Because a null value is nothing, it never equals anything in a search condition, so it never evaluates as being true.

##### Try it now

Execute any or all of the three preceding queries, and see that they don't return any matching rows.

### 7.2.2 How to search for null values correctly

To search for null values correctly, let's state another verbal example for what we're trying to do. If we want the full name of any author who has a null value for a middle name, we could say the following: "I would like the first, middle, and last name of authors where the middle name is null."

To turn previous verbal declarations like this one into SQL queries, we replaced *is* with the `=` operator. But because we're dealing with a filter condition involving null values, which don't work with comparison operators, we can instead use a new operator that is literally the last two words of the verbal declaration: `IS` `NULL`. Here's what our query will look like, with the results shown in figure 7.8:

```sql
SELECT
    FirstName,
    MiddleName,
    LastName
FROM author
WHERE MiddleName IS NULL;
```

##### Figure 7.8 The results of first, middle, and last names in the author table for authors with no middle name

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH07_F08_Iannucci.png)

Pay close attention to null values and the `IS` `NULL` operator because null values can be even more problematic. Notice that in figure 7.7, the first of the 11 rows in the author table has a MiddleName value of K. Suppose that you want to query all rows except that one, with an exclusion query (chapter 6). You could do this with the following query, with the results shown in figure 7.9:

```sql
SELECT
    FirstName,
    MiddleName,
    LastName
FROM author
WHERE MiddleName <> 'K';
```

##### Figure 7.9 The rows returned for any author that does not have a middle name of K, which excludes any author without a middle name

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH07_F09_Iannucci.png)

You may think this query would return the 10 rows that don't have K as a middle name, but you'd be incorrect. Because the filter is looking for any value that does not equal K, it discards any results that have null values. Nothing cannot equal (or even not equal) something, so the filter considers only rows that have a non-null value for MiddleName.

It's possible, of course, that you intend this query to return only rows with a value for MiddleName, but if you want all 10 rows to be returned, you need to include the `IS` `NULL` operator in your filtering. You'd use an additional `OR` operator, with the results shown in figure 7.10:

```sql
SELECT
    FirstName,
    MiddleName,
    LastName
FROM author
WHERE MiddleName <> 'K'
    OR MiddleName IS NULL;
```

##### Figure 7.10 All rows that don't have a middle name of K are returned, including those with null values

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH07_F10_Iannucci.png)

### 7.2.3 How to search for values that are not null

Now that we've learned how to include rows with null values, let's look at how to return all rows that do not have a null value. We can start once again by declaring verbally what we want: "I would like the first, middle, and last names of authors where the middle name isn't null."

The word *isn't* is a contraction for *is not,* of course, which is exactly how our next operator will work, shown in the following query and figure 7.11:

```sql
SELECT
    FirstName,
    MiddleName,
    LastName
FROM author
WHERE MiddleName IS NOT NULL;
```

##### Figure 7.11 The results for all rows from the author table with no middle name

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH07_F11_Iannucci.png)

The `IS` `NOT` `NULL` operator allows us to return all rows with some value other than NULL for a given column. Interestingly enough, we can get the same results using a wildcard we learned about earlier in this chapter:

```sql
SELECT
FirstName,
    MiddleName,
    LastName
FROM author
WHERE MiddleName LIKE '%';
```

Why does this query work the same way as when we use the `IS` `NOT` `NULL` operator? The `%` wildcard matches any string of data so long as there is data in the column. Because null values have no data, wildcards never match them or return them in a result set, which is essentially what the `IS NOT NULL` operator also does.

##### Try it now

Execute the preceding two queries using `IS` `NOT` `NULL` and the `%` wildcard, and see that the results are the same as in figure 7.11.

All right—we've spent three chapters examining a multitude of ways to filter results when querying a table. In chapter 8, we'll level up your SQL knowledge even more by learning how to query multiple tables at the same time.

## 7.3 Lab

Let's take a moment to review the ways you've learned to filter rows. Write some SQL queries to find the following:

1.  The full names of all authors who have a middle name of Anne or no middle name at all

2.  The full names of all authors who have no middle name and have a first name that starts with *D*

3.  The title name and price of all titles that start with the word *The* and have a price less than $10.00

4.  The title name and publication date of any title that ends with *S* and was published after January 1, 2020

5.  The title name of any title containing the word *of* or the word *in*

## 7.4 Lab answers

1.  The answer is

```sql
SELECT
    FirstName,
    MiddleName,
    LastName
FROM author
WHERE MiddleName = 'Anne'
    OR MiddleName IS NULL;
```

2.  The answer is

```sql
SELECT
    FirstName,
    MiddleName,
    LastName
FROM author
WHERE FirstName LIKE 'D%'
    AND MiddleName IS NULL;
```

3.  The answer is

```sql
SELECT
    TitleName,
    Price
FROM title
WHERE TitleName LIKE 'The%'
    AND Price < 10;
```

4.  The answer is

```sql
SELECT
    TitleName,
    PublicationDate
FROM title
WHERE TitleName LIKE '%s'
    AND PublicationDate > '2020-01-01';
```

5.  This question is a bit of a trick question designed to challenge you. You have to consider that depending on how you write the query, you might get more or less data than you want. You might write something as simple as this:

```sql
SELECT
    TitleName
FROM title
WHERE TitleName LIKE '%of%'
    OR TitleName LIKE '%in%';
```

If you execute that query, you get not only The Call of the While, Anne of Fact Tables, and Catcher in the Try—all of which meet the requirement—but also The Join Luck Club and The DateTime Machine, which don't. The latter two are included in the results because they have a string value matching the value of the letters *in* within their title names.

So how can we exclude those undesired results? One way is to add leading and trailing spaces to the strings we're searching for, like this:

```sql
SELECT
    TitleName
FROM title
WHERE TitleName LIKE '% of %'
    OR TitleName LIKE '% in %';
```

This query returns the desired results but may not work in a different scenario. Because now we're looking at strings involving leading or trailing spaces, we won't have any results if the title names start or end with the word *in* or *of.* To include any hypothetical results that started or ended with the words we were searching for, we need to include additional conditions:

```sql
SELECT
    TitleName
FROM title
WHERE TitleName LIKE '% of %'
    OR TitleName LIKE '% in %'
    OR TitleName LIKE 'of %'
    OR TitleName LIKE 'in %'
    OR TitleName LIKE '% of'
    OR TitleName LIKE '% in';
```

Again, this question is admittedly difficult, but it's designed to get you thinking about the possible values of data and how to use what you've learned so far creatively to write an accurate query.

