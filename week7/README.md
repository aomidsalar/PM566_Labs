---
title: "Lab 7"
author: "Audrey Omidsalar"
date: "10/8/2021"
output:
  html_document:
    toc: yes
    toc_float: yes
    keep_md: yes
  github_document:
  always_allow_html: true
---



## Question 1: How many SARS-CoV2 papers?

```r
# Downloading the website
website <- read_html(x = "https://pubmed.ncbi.nlm.nih.gov/?term=sars-cov-2")

# Finding the counts
counts <- xml2::xml_find_first(website, "/html/body/main/div[9]/div[2]/div[2]/div[1]")

# Turning it into text
counts <- as.character(counts)

# Extracting the data using regex
numberpapers  <- stringr::str_extract(counts, "[:digit:]+.*[:digit:]")
```

#### There are 114,592 papers about SARS-CoV2 on PubMed

## Question 2: Academic publications on COVID19 and Hawaii


```r
query_ids <- GET(
  url   = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi",
  query = list(
    db = "pubmed",
    term = "covid19 hawaii",
    retmax = 1000
  )
)

# Extracting the content of the response of GET
ids <- httr::content(query_ids)
```

## Question 3

```r
#ids_list <- xml2::as_list(ids)
# Turn the result into a character vector
ids <- as.character(ids)

# Find all the ids 
ids <- stringr::str_extract_all(ids, "<Id>[:digit:]+</Id>")[[1]]

# Remove all the leading and trailing <Id> </Id>. Make use of "|"
ids <- stringr::str_remove_all(ids, "<Id>|</Id>")
#ids <- stringr::str_remove_all(ids, "<?/Id>")
```


```r
publications <- GET(
  url   = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi",
  query = list(
    db = "pubmed",
    id = I(paste(ids, collapse = ",")),
    retmax = 1000,
    rettype = "abstract"
    )
)

# Turning the output into character vector
publications <- httr::content(publications)
publications_txt <- as.character(publications)
```


