---
title: "Lab 10"
author: "Audrey Omidsalar"
date: "11/5/2021"
output:
  html_document:
    toc: yes
    toc_float: yes
    keep_md: yes
  github_document:
  always_allow_html: true
---



## Setup

```r
# Initialize a temporary in memory database
con <- dbConnect(SQLite(), ":memory:")
# Download tables
actor <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/actor.csv")
rental <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/rental.csv")
customer <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/customer.csv")
payment <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/payment_p2007_01.csv")

# Copy data.frames to database
dbWriteTable(con, "actor", actor)
dbWriteTable(con, "rental", rental)
dbWriteTable(con, "customer", customer)
dbWriteTable(con, "payment", payment)
```
List tables

```r
dbListTables(con)
```

```
## [1] "actor"    "customer" "payment"  "rental"
```
#### Getting information about the *actor* table

```r
dbGetQuery(con ,"PRAGMA table_info(actor)")
```

```
##   cid        name    type notnull dflt_value pk
## 1   0    actor_id INTEGER       0         NA  0
## 2   1  first_name    TEXT       0         NA  0
## 3   2   last_name    TEXT       0         NA  0
## 4   3 last_update    TEXT       0         NA  0
```

## Exercise 1

Retrieve the actor ID, first name and last name for all actors using the actor table. Sort by last name and then by first name. (Using LIMIT (head() in R) to output the top 5)


```r
dbGetQuery(con , "SELECT actor_id, first_name, last_name
FROM actor
ORDER by last_name, first_name /* comments can be added
on multiple lines like this */
LIMIT 5")
```

```
##   actor_id first_name last_name
## 1       58  CHRISTIAN    AKROYD
## 2      182     DEBBIE    AKROYD
## 3       92    KIRSTEN    AKROYD
## 4      118       CUBA     ALLEN
## 5      145        KIM     ALLEN
```

## Exercise 2
Retrieve the actor ID, first name, and last name for actors whose last name equals ‘WILLIAMS’ or ‘DAVIS’.
Using LIMIT to output the top 5


```r
dbGetQuery(con, "SELECT actor_id, first_name, last_name
FROM actor
WHERE last_name IN ('WILLIAMS', 'DAVIS')")
```

```
##   actor_id first_name last_name
## 1        4   JENNIFER     DAVIS
## 2       72       SEAN  WILLIAMS
## 3      101      SUSAN     DAVIS
## 4      110      SUSAN     DAVIS
## 5      137     MORGAN  WILLIAMS
## 6      172    GROUCHO  WILLIAMS
```

## Exercise 3
Write a query against the rental table that returns the IDs of the customers who rented a film on July 5, 2005 (use the rental.rental_date column, and you can use the date() function to ignore the time component). Include a single row for each distinct customer ID.


```r
dbGetQuery(con, "SELECT DISTINCT customer_id
FROM rental
WHERE date(rental_date)= '2005-07-05'
")
```

```
##    customer_id
## 1          565
## 2          242
## 3           37
## 4           60
## 5          594
## 6            8
## 7          490
## 8          476
## 9          322
## 10         298
## 11         382
## 12         138
## 13         520
## 14         536
## 15         114
## 16         111
## 17         296
## 18         586
## 19         349
## 20         397
## 21         369
## 22         421
## 23         142
## 24         169
## 25         348
## 26         553
## 27         295
```

## Exercise 4.1
Construct a query that retrieves all rows from the payment table where the amount is either 1.99, 7.99, 9.99.


```r
q <- dbSendQuery(con, "SELECT *
FROM payment
WHERE amount IN (1.99, 7.99, 9.99)")
dbFetch(q, n=10)
```

```
##    payment_id customer_id staff_id rental_id amount               payment_date
## 1       16050         269        2         7   1.99 2007-01-24 21:40:19.996577
## 2       16056         270        1       193   1.99 2007-01-26 05:10:14.996577
## 3       16081         282        2        48   1.99 2007-01-25 04:49:12.996577
## 4       16103         294        1       595   1.99 2007-01-28 12:28:20.996577
## 5       16133         307        1       614   1.99 2007-01-28 14:01:54.996577
## 6       16158         316        1      1065   1.99 2007-01-31 07:23:22.996577
## 7       16160         318        1       224   9.99 2007-01-26 08:46:53.996577
## 8       16161         319        1        15   9.99 2007-01-24 23:07:48.996577
## 9       16180         330        2       967   7.99 2007-01-30 17:40:32.996577
## 10      16206         351        1      1137   1.99 2007-01-31 17:48:40.996577
```

```r
##fetch query to get results in 10-row chunks
dbClearResult(q)
```

## Exercise 4.2
Construct a query that retrieves all rows from the payment table where the amount is greater then 5


```r
dbGetQuery(con, "SELECT *
      FROM payment
      WHERE amount > 5
      LIMIT 5")
```

```
##   payment_id customer_id staff_id rental_id amount               payment_date
## 1      16052         269        2       678   6.99 2007-01-28 21:44:14.996577
## 2      16058         271        1      1096   8.99 2007-01-31 11:59:15.996577
## 3      16060         272        1       405   6.99 2007-01-27 12:01:05.996577
## 4      16061         272        1      1041   6.99 2007-01-31 04:14:49.996577
## 5      16068         274        1       394   5.99 2007-01-27 09:54:37.996577
```
How many of these records are there?

```r
dbGetQuery(con, "SELECT COUNT (*)
      FROM payment
      WHERE amount > 5
      LIMIT 5")
```

```
##   COUNT (*)
## 1       266
```

How many per staff_id?

```r
dbGetQuery(con, "SELECT staff_id, COUNT (*) as N
      FROM payment
      WHERE amount > 5
      GROUP BY staff_id
      LIMIT 5")
```

```
##   staff_id   N
## 1        1 151
## 2        2 115
```

## Exercise 4.3
Construct a query that retrieves all rows from the payment table where the amount is greater then 5 and less then 8


```r
dbGetQuery(con, "SELECT *
      FROM payment
      WHERE amount > 5 AND amount < 8
      LIMIT 5")
```

```
##   payment_id customer_id staff_id rental_id amount               payment_date
## 1      16052         269        2       678   6.99 2007-01-28 21:44:14.996577
## 2      16060         272        1       405   6.99 2007-01-27 12:01:05.996577
## 3      16061         272        1      1041   6.99 2007-01-31 04:14:49.996577
## 4      16068         274        1       394   5.99 2007-01-27 09:54:37.996577
## 5      16074         277        2       308   6.99 2007-01-26 20:30:05.996577
```

## Exercise 5
Retrieve all the payment IDs and their amount from the customers whose last name is ‘DAVIS’.


```r
dbGetQuery(con, "SELECT p.payment_id, p.amount
FROM payment as p
  INNER JOIN customer as c
  ON p.customer_id = c.customer_id
  WHERE c.last_name = 'DAVIS'")
```

```
##   payment_id amount
## 1      16685   4.99
## 2      16686   2.99
## 3      16687   0.99
```

## Exercise 6.1
Use COUNT(*) to count the number of rows in rental


```r
dbGetQuery(con, "SELECT COUNT (*) as Number_Rows
    FROM rental")
```

```
##   Number_Rows
## 1       16044
```

## Exercise 6.2
Use COUNT(*) and GROUP BY to count the number of rentals for each customer_id


```r
dbGetQuery(con, "SELECT customer_id, COUNT(*) as Number_Rentals
        FROM rental
        GROUP BY customer_id
        LIMIT 10")
```

```
##    customer_id Number_Rentals
## 1            1             32
## 2            2             27
## 3            3             26
## 4            4             22
## 5            5             38
## 6            6             28
## 7            7             33
## 8            8             24
## 9            9             23
## 10          10             25
```

## Exercise 6.3
Repeat the previous query and sort by the count in descending order


```r
dbGetQuery(con, "SELECT customer_id, COUNT(*) as Number_Rentals
        FROM rental
        GROUP BY customer_id
        ORDER BY Number_Rentals DESC
        LIMIT 10")
```

```
##    customer_id Number_Rentals
## 1          148             46
## 2          526             45
## 3          236             42
## 4          144             42
## 5           75             41
## 6          469             40
## 7          197             40
## 8          468             39
## 9          178             39
## 10         137             39
```

## Exercise 6.4
Repeat the previous query but use HAVING to only keep the groups with 40 or more.


```r
dbGetQuery(con, "SELECT customer_id, COUNT(*) as Number_Rentals
        FROM rental
        GROUP BY customer_id
        HAVING Number_Rentals >= 40
        ORDER BY Number_Rentals DESC")
```

```
##   customer_id Number_Rentals
## 1         148             46
## 2         526             45
## 3         236             42
## 4         144             42
## 5          75             41
## 6         469             40
## 7         197             40
```

## Exercise 7
Calculate max, min, avg, and sum for the payment table


```r
dbGetQuery(con, "SELECT MAX(amount) as max_amount,
    MIN(amount) as min_amount,
    AVG(amount) as avg_amount,
    SUM(amount) as sum_amount
    FROM payment
    LIMIT 10")
```

```
##   max_amount min_amount avg_amount sum_amount
## 1      11.99       0.99   4.169775    4824.43
```

## Exercise 7.1
Modify the above query to do those calculations for each customer_id


```r
dbGetQuery(con, "SELECT customer_id,
    MAX(amount) as max_amount,
    MIN(amount) as min_amount,
    AVG(amount) as avg_amount,
    SUM(amount) as sum_amount
    FROM payment
    GROUP BY customer_id
    LIMIT 10")
```

```
##    customer_id max_amount min_amount avg_amount sum_amount
## 1            1       2.99       0.99   1.990000       3.98
## 2            2       4.99       4.99   4.990000       4.99
## 3            3       2.99       1.99   2.490000       4.98
## 4            5       6.99       0.99   3.323333       9.97
## 5            6       4.99       0.99   2.990000       8.97
## 6            7       5.99       0.99   4.190000      20.95
## 7            8       6.99       6.99   6.990000       6.99
## 8            9       4.99       0.99   3.656667      10.97
## 9           10       4.99       4.99   4.990000       4.99
## 10          11       6.99       6.99   6.990000       6.99
```

## Exercise 7.2
Modify the above query to only keep the customer_ids that have more then 5 payments


```r
dbGetQuery(con, "SELECT customer_id,
    COUNT (*) as number_payments,
    MAX(amount) as max_amount,
    MIN(amount) as min_amount,
    AVG(amount) as avg_amount,
    SUM(amount) as sum_amount
    FROM payment
    GROUP BY customer_id
    HAVING COUNT(*) > 5
    LIMIT 10")
```

```
##    customer_id number_payments max_amount min_amount avg_amount sum_amount
## 1           19               6       9.99       0.99   4.490000      26.94
## 2           53               6       9.99       0.99   4.490000      26.94
## 3          109               7       7.99       0.99   3.990000      27.93
## 4          161               6       5.99       0.99   2.990000      17.94
## 5          197               8       3.99       0.99   2.615000      20.92
## 6          207               6       6.99       0.99   2.990000      17.94
## 7          239               6       7.99       2.99   5.656667      33.94
## 8          245               6       8.99       0.99   4.823333      28.94
## 9          251               6       4.99       1.99   3.323333      19.94
## 10         269               6       6.99       0.99   3.156667      18.94
```

## Clean-up
Run the following chunk to disconnect from the connection.


```r
dbDisconnect(con)
```

