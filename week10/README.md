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

```r
#dbGetQuery(con , "SELECT actor ID, first name, last name
#FROM actor
#ORDER by last name, first name")
```


