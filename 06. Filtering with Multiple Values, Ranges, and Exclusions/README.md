# 6 Filtering with Multiple Values, Ranges, and Exclusions

As we saw in chapter 5, the `WHERE` clause offers many useful options for filtering results based on specific conditions. We looked at several examples of filtering on a single value using the `AND` and `OR` operators. Now we'll expand that concept to filter on even more values, including a list of specific values or ranges of unspecified values.

These are examples of *positive searches*, in which we try to match values that we want to see in the results of our queries. Because often, we'll want to do the opposite and see all the values except some specific filter conditions, we'll also see how to negate any of the conditions we've covered. Let's start by looking at a new operator for the `WHERE` clause.

## 6.1 Filtering on specific values

Previously, we looked at a basic search, finding the TitleNames for titles that had a certain Price. If we want to query titles that had a Price of $10.95, for example, we'd write our SQL like this:

```sql
SELECT
    TitleName,
    Price
FROM title
WHERE Price = 10.95;
```

But what if we want to find the titles with a Price of either $10.95 or $12.95? Now we know that we can write SQL to do this using the `OR` operator, so we could write a SQL query like this (output shown in figure 6.1):

```sql
SELECT
    TitleName,
    Price
FROM title
WHERE Price = 10.95
    OR Price = 12.95;
```

##### Figure 6.1 The results of a query with filter conditions for a Price of 10.95 or 12.95

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH06_F01_Iannucci.png)

This query will give us the results we want for our two filter condition values, but it could lead to some wordy SQL if we have a list of conditions that grows much longer. We don't want to use an `OR` operator if we have 3, 10, or even more filter condition values for a single column.

To resolve this problem, SQL has the `IN` operator, which allows us to consolidate our filter conditions into a single operator. The `IN` operator has three requirements:

* The list of values must be comma-delimited.
* The list of values must be enclosed in parentheses.
* The requirement of single quotes is the same as in any other filter condition.

As an example, we can rewrite the preceding query with an `IN` operator, like this:

```sql
SELECT
    TitleName,
    Price
FROM title
WHERE Price IN (10.95, 12.95);
```

Executing this query yields the results shown in figure 6.1 but in a more compact form of SQL. As I mentioned earlier, the use of single quotes is the same as what we used in queries that filtered for filter conditions with string or date values.

##### Try it now

Using the `IN` operator, execute a query looking for TitleName and Price from titles where Price is $7.95, $8.95, or $9.95.

If we want to search for titles with a specific PublicationDate, we need to use single quotes with our filter conditions because we're using data values instead of numeric values. We'd write our SQL like this (results shown in figure 6.2):

```sql
SELECT
    TitleName,
    PublicationDate
FROM title
WHERE PublicationDate IN ('2015-04-30', '2016-02-06');
```

##### Figure 6.2 The results for titles with a PublicationDate of 2015-04-30 or 2016-02-06

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH06_F02_Iannucci.png)

##### Note

As far as the relational database management system (RDBMS) is concerned, the order of values used with the `IN` operator is irrelevant. As with any query that doesn't involve an `ORDER BY` clause, the rows in the result set aren't guaranteed to be returned in any particular order. That said, as I've noted several times in this book, you should follow best practices to make your SQL more human-readable whenever possible. For this reason, it's best to list values in the filter conditions of the `IN` operator in numerical, alphabetical, or chronological order.

## 6.2 Filtering on a range of values

Filtering on a list of specific values with the `IN` operator is useful if you know all the specific values you want to match, but not when you don't know all the specific values. Often, you need to find values with a range, such as any values higher than a certain amount or older than a certain date. You can use *comparison operators* in SQL, which compare two values to see whether they meet specific criteria, to find the desired results.

### 6.2.1 Filtering on an open-ended range

Assuming that you've worked with basic math, you're familiar with the *less-than* (`<`) and *greater-than* (`>`) signs. (On a typical keyboard, they share the same keys as the comma and the period, respectively.) Like the equal sign (=), these signs are commonly used comparison operators. Unlike the equal sign, the less-than and greater-than signs can be used to find an open-ended range of values.

In a query earlier in this chapter, we looked for any title that had a Price of $10.95 or $12.95. In our title table, these titles are the ones with the highest prices. We can find these same two titles by writing a query using the `>` sign. Let's also include an `ORDER BY` clause to see the Price values in order, recalling that the default is to order the results by ascending values. Figure 6.3 shows the output of this query:

```sql
SELECT
    TitleName,
    Price
FROM title
WHERE Price > 9.95
ORDER BY Price ASC;
```

##### Figure 6.3 The titles that have a Price greater than 9.95, ordered by Price ascending

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH06_F03_Iannucci.png)

We can use the `<` sign to do the opposite: find any titles that have a Price of less than $9.95. For this query, we'll sort by descending Price values to show the prices closest to $9.95 at the top of the results (shown in figure 6.4):

```sql
SELECT
    TitleName,
    Price
FROM title
WHERE Price < 9.95
ORDER BY Price DESC;
```

##### Figure 6.4 The titles that have a Price less than 9.95, ordered by Price descending

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH06_F04_Iannucci.png)

You've probably noticed that neither of these result sets includes any titles that match the Price of $9.95. Certainly, you'll sometimes want to query a range of values including those that match the filter conditions, so SQL also includes the comparison operators *less-than or equal-to* (<=) and *greater-than or equal-to* (>=). Using one of these operators, we can query any title with a Price greater than or equal to a filter condition of `9.95` and get the results shown in figure 6.5:

```sql
SELECT
    TitleName,
    Price
FROM title
WHERE Price >= 9.95
ORDER BY Price ASC;
```

##### Figure 6.5 The titles that have a Price greater than or equal to 9.95, ordered by Price ascending

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH06_F05_Iannucci.png)

##### Note

You can see how we can search for values greater than or less than a particular numeric or date value. Although the use cases are less common, you can also use these operators with string values. The values are typically returned with respect to precedence in the alphabet. Be careful when using these operators with string values, however: the way that the numeric, symbol, and case values are filtered is based on the collation settings in your RDBMS. *Collation* determines sorting rules based on character qualities such as the character set, letter case, and accent use; therefore, different collations return characters in a different order.

### 6.2.2 Filtering a defined range

The operators in section 6.2.1 are *open-ended*—that is, they could include an infinite number of values greater than or less than a value. If we want to find values greater than some value but lesser than another value, we have a couple of options.

You may have already thought of the first option, which is to use both `>` and `<` in the `WHERE` clause. If you're trying to find titles with a price between $8.95 and $10.95, you can write a SQL query like the following. Again, you'll use an `ORDER BY` in the query to organize the results so that you can see the maximum and minimum values returned by your filter conditions in figure 6.6:

```sql
SELECT
    TitleName,
    Price
FROM title
WHERE Price > 8.95
    AND Price < 10.95
ORDER BY Price ASC;
```

##### Figure 6.6 The titles with Price values between the search conditions of 8.95 and 10.95

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH06_F06_Iannucci.png)

We can see that the only Price value that matches the range of our filter conditions is 9.95. If we want to change the filter conditions to *include* the values we've specified, we need to use `>=` and `<=` in the `WHERE` clause like this:

```sql
SELECT
    TitleName,
    Price
FROM title
WHERE Price >= 8.95
    AND Price <= 10.95
ORDER BY Price ASC;
```

As figure 6.7 shows, now the result set includes titles that match the Price values used in the filter conditions.

##### Figure 6.7 The titles with Price values between the filter conditions of 8.95 and 10.95 when the values used in the filter condition are included

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH06_F07_Iannucci.png)

We also have another way to search a range of values: instead of using `>=` and `<=`, we can use the `BETWEEN` operator to perform the same function. The rules for using `BETWEEN` are

* The column being searched is mentioned only once.
* Only two filter conditions can be supplied.
* The first value represents the low end of the range, and the second value represents the high end of the range.
* The filter conditions must be separated by the word `AND`.
* Any values matching either condition are included in the results.

We can write the previous query using the `BETWEEN` operator like this:

```sql
SELECT
    TitleName,
    Price
FROM title
WHERE Price BETWEEN 8.95 AND 10.95
ORDER BY Price ASC;
```

Even though we used one less line of SQL, the results of this query should be identical to the results shown in figure 6.7.

##### Warning

I should mention again that when you're using `BETWEEN`, the first value represents the low end of the range, and the second represents the high end of the range. If you use `BETWEEN`, and the first value of the filter conditions of the range is higher than the second value, your query will return no results. The SQL query `WHERE` `Price` `BETWEEN` `10.95` `and` `8.95`, for example, won't return any rows, regardless of other filter conditions. Logically, no values are both higher than `10.95` and lower than `8.95`.

## 6.3 Negating filter conditions

So far, we've used the `WHERE` clause to specify filter conditions that match specific values or ranges of data. These conditions are known as *inclusive* because the results include rows with values that match the values we used in our filter conditions. The opposite conditions are known as *exclusive*; we want all values *except* the ones specified in the filter conditions. In a way, this is a bit like the way we used `OFFSET` in chapter 4 to exclude a specific number of rows, but we're being much more specific about what we're excluding.

### 6.3.1 Negating a specific value

In chapter 5, we learned to filter on specific values using the equal sign (`=`). Mathematically, we can represent the opposite using the *not-equal sign* (`<>`), which combines the `<` and `>` signs.

If we want to list all titles that don't have a Price of $7.95, we could use `<>` like this, with the ordered results shown in figure 6.8:

```sql
SELECT
    TitleName,
    Price
FROM title
WHERE Price <> 7.95
ORDER BY Price ASC;
```

##### Figure 6.8 All titles, excluding the two that have a Price of 7.95

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH06_F08_Iannucci.png)

Even though we've used `<>` for mathematical comparisons, we can use it with date and string values as well. If we want all titles *except* those published on February 6, 2016, we can use single quotes with `<>` to get the results shown in figure 6.9:

```sql
SELECT
    TitleName,
    PublicationDate
FROM title
WHERE PublicationDate <> '2016-02-06'
ORDER BY PublicationDate ASC;
```

##### Figure 6.9 All titles, excluding the one ("The Join Luck Club") that has a PublicationDate of 2016-02-06

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH06_F09_Iannucci.png)

##### Note

The RDBMS you use may offer the option to use `!=` instead of `<>`. Both options perform the same function of negating a single condition. But I advise you to develop the habit of using `<>` in your SQL because not every RDBMS supports the `!=` option.

### 6.3.2 Negating any filter condition

Although `<>` enables us to exclude a single value in a filter condition, one operator excludes an entire filter condition. This operator, `NOT`, turns any inclusive filter condition into an exclusive condition.

In the preceding query, for example, we used `<>` to exclude any title that has a PublicationDate of February 6, 2016. We can use the opposite of `<>`, which is `=`, with the `NOT` operator to achieve the same results. The following query produces the results shown in figure 6.9, with the results ordered by PublicationDate:

```sql
SELECT
    TitleName,
    PublicationDate
FROM title
WHERE NOT PublicationDate = '2016-02-06'
ORDER BY PublicationDate ASC;
```

In this case, the `NOT` operator immediately follows the word `WHERE`. You can use the `NOT` operator after the start of any filter condition to negate it, which means you could also use it immediately after conditions that begin with the `AND` or `OR` operator.

##### Try it now

Execute the two preceding queries to see how to use `<>` and `NOT` to exclude specific values.

You shouldn't use the negative `NOT` operator with `>` or `<` because you can logically use their positive opposites (`<=` and `>=`, respectively) to get the same results with simpler syntax. it's not uncommon, however, to use `NOT` with the `IN` operator mentioned in section 6.1.

Recall that we used a single condition with the `IN` operator to query all titles with a Price of $10.95 or $12.95 to replace two conditions using the `OR` operator. We can use `NOT` to negate this filter condition with the `IN` operator, returning all titles *except* those that match the conditions of a Price of $10.95 or $12.95, with the results sorted by Price (figure 6.10):

```sql
SELECT
    TitleName,
    Price
FROM title
WHERE NOT Price IN (10.95, 12.95)
ORDER BY Price ASC;
```

##### Figure 6.10 All titles, excluding those with a Price of 10.95 or 12.95

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH06_F10_Iannucci.png)

You may have noticed the condition `WHERE` `NOT` `Price` `IN` in the preceding query, which sounds a bit clumsy to English-speaking people. There's a better and more common way to get the same results in SQL. There is a separate `NOT` `IN` operator, so we can also place `NOT` before the `IN` in our filter condition to make it more readable, like this:

```sql
SELECT TitleName, Price
FROM title
WHERE Price NOT IN (10.95, 12.95)
ORDER BY Price ASC;
```

This query also returns the results shown in figure 6.10. When we want to use an exclusionary list of values, as in the preceding example, `NOT` `IN` is more commonly used.

## 6.4 Combining types of filter conditions

This chapter has shown you several new ways to filter your data with inclusive and exclusive filter conditions. One last point to make is that you can (and will) combine both kinds of filter conditions in your queries. You could write a query, for example, to find any title that has an Advance value > 5000 but a Royalty <> 12%, with the results shown in figure 6.11:

```sql
SELECT
    TitleName,
    Advance,
    Royalty
FROM title
WHERE Advance > 5000
    AND Royalty <> 12;
```

##### Figure 6.11 The titles that have an Advance greater than 5000 with a Royalty that is not 12 (%)

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH06_F11_Iannucci.png)

With all the operators discussed in this chapter and chapter 5, you can begin to write some relatively complex filter conditions. If you want to include the preceding results (any title that has an Advance > 5000 but a Royalty <> 12 [%]) and also any titles published after January 1, 2020, you can easily do that by using the SQL for inclusive and exclusive queries and controlling the logic with parentheses. If you tried this approach, your SQL query might look something like the following, with the results shown in figure 6.12:

```sql
SELECT
    TitleName,
    Advance,
    Royalty,
    PublicationDate
FROM title
WHERE (Advance > 5000
    AND Royalty <> 12)
    OR (PublicationDate > '2020-01-01');
```

##### Figure 6.12 The titles with an Advance of 5000 that don't have a Royalty of 12 (%) or any title published after 2020-01-01

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH06_F12_Iannucci.png)

##### Tip

Although you can use exclusive filter conditions to achieve the same results as inclusive filter conditions, it's preferable to use inclusive filter conditions whenever possible. Generally, inclusive conditions are easier for others who read your SQL to comprehend and are processed more efficiently by the RDBMS. An exception would occur if you had only one exclusion or a few exclusions to filter, in which case it would likely be better to use exclusive filters instead of an inclusive filter with a vast number of inclusive conditions or values.

You're only a few chapters into this book, but already, you can search data in meaningful, accurate ways. You've used mostly filter conditions with numeric and date values so far. But chapter 7 discusses another series of tools you can use in the `WHERE` clause to perform advanced searches for data in string values.

## 6.5 Reviewing comparison operators

This chapter covered more than a dozen comparison operators that you can use for filtering in the `WHERE` clause. You may have been taking lots of notes, but in case you didn't, table 6.1 presents a list of what you used.

##### Table 6.1 Review of `WHERE` clause comparison operators [(view table figure)](https://drek4537l1klr.cloudfront.net/iannucci/HighResolutionFigures/table_6-1.png)

| Operator | Description |
| --- | --- |
| `=` | Equality |
| `<>` | Inequality |
| `!=` | Inequality* |
| `<` | Less than |
| `>` | Greater than |
| `!<` | Not less than* |
| `!>` | Not greater than* |
| `<=` | Less than or equal to |
| `>=` | Greater than or equal to |
| `BETWEEN` | Between two values, including those values |
| `IN` | Equality to a list of multiple values |
| `NOT IN` | Inequality to a list of multiple values |
| `NOT` | Inequality to stated condition |
| * May not be supported by every RDBMS | |

## 6.6 Lab

You have only one lab assignment today, but it is a challenge that uses your creativity. This chapter and chapter 5 covered quite a few ways to include and exclude data, so consider all that you've learned about using the `WHERE` clause for filtering.

For this exercise, think of as many ways as possible to use the `WHERE` clause to return the TitleName and Price for all rows in the title table that don't have a price of 9.95. The results of each query should include only the rows shown in figure 6.13, ordered by Price.

##### Figure 6.13 The titles with a Price that is not 9.95, ordered by Price ascending

![](https://drek4537l1klr.cloudfront.net/iannucci/Figures/CH06_F13_Iannucci.png)

## 6.7 Lab answers

These are some of the many ways you can write the `WHERE` clause to exclude titles with a Price of $9.95:

* `WHERE Price <> 9.95`
* `WHERE NOT Price = 9.95`
* `WHERE Price < 9.95 OR Price > 9.95`
* `WHERE PRICE NOT IN (9.95)`
* `WHERE Price IN (7.95, 8.95, 10.95, 12.95)`
* `WHERE Price BETWEEN 7.95 AND 8.95 OR Price BETWEEN 10.95 and 12.95`
* `WHERE NOT Price BETWEEN 9.95 and 9.95`

That last answer may be a bit unexpected because it effectively negates a range of values that include only a single value. I show it here only to demonstrate that a range with the same high and low end can be executed.

