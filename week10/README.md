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

Retrieve the actor ID, first name and last name for all actors using the actor table. Sort by last name and then by first name. Use LIMIT (head() in R) to output the top 5.


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


```r
dbGetQuery(con, "SELECT COUNT (*)
    FROM rental")
```

```
##   COUNT (*)
## 1     16044
```

## Exercise 6.2


```r
dbGetQuery(con, "SELECT customer_id, COUNT(*)
        FROM rental
        GROUP BY customer_id
        LIMIT 10")
```

```
##    customer_id COUNT(*)
## 1            1       32
## 2            2       27
## 3            3       26
## 4            4       22
## 5            5       38
## 6            6       28
## 7            7       33
## 8            8       24
## 9            9       23
## 10          10       25
```

## Exercise 6.3


```r
dbGetQuery(con, "SELECT customer_id, COUNT(*)
        FROM rental
        GROUP BY customer_id
        ORDER BY COUNT(*) DESC
        LIMIT 10")
```

```
##    customer_id COUNT(*)
## 1          148       46
## 2          526       45
## 3          236       42
## 4          144       42
## 5           75       41
## 6          469       40
## 7          197       40
## 8          468       39
## 9          178       39
## 10         137       39
```

## Exercise 6.4


```r
dbGetQuery(con, "SELECT customer_id, COUNT(*)
        FROM rental
        GROUP BY customer_id
        HAVING COUNT(*) > 40
        ORDER BY COUNT(*) DESC")
```

```
##   customer_id COUNT(*)
## 1         148       46
## 2         526       45
## 3         236       42
## 4         144       42
## 5          75       41
```

## Exercise 7



