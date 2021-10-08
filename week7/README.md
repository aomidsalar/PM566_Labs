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

##Question 2: Academic publications on COVID19 and Hawaii




