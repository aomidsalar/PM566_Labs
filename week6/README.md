---
title: "Lab 6"
author: "Audrey Omidsalar"
date: "10/1/2021"
output:
  html_document:
    toc: yes
    toc_float: yes
    keep_md: yes
  github_document:
  always_allow_html: true
---



### Read in Data

```r
fn <- "mtsamples.csv"
if (!file.exists(fn))
  download.file("https://raw.githubusercontent.com/USCbiostats/data-science-data/master/00_mtsamples/mtsamples.csv", destfile = fn)
mtsamples <- read.csv(fn)
##convert data frame into tibble
mtsamples <- as_tibble(mtsamples)
head(mtsamples)
```

```
## # A tibble: 6 × 6
##       X description    medical_specialty sample_name  transcription   keywords  
##   <int> <chr>          <chr>             <chr>        <chr>           <chr>     
## 1     0 " A 23-year-o… " Allergy / Immu… " Allergic … "SUBJECTIVE:, … "allergy …
## 2     1 " Consult for… " Bariatrics"     " Laparosco… "PAST MEDICAL … "bariatri…
## 3     2 " Consult for… " Bariatrics"     " Laparosco… "HISTORY OF PR… "bariatri…
## 4     3 " 2-D M-Mode.… " Cardiovascular… " 2-D Echoc… "2-D M-MODE: ,… "cardiova…
## 5     4 " 2-D Echocar… " Cardiovascular… " 2-D Echoc… "1.  The left … "cardiova…
## 6     5 " Morbid obes… " Bariatrics"     " Laparosco… "PREOPERATIVE … "bariatri…
```

```r
##piping it to knitr::kable() makes the table look pretty
```
## Question 1
### What specialties do we have?
There are 40 specialties in total within this dataset. The specialties do not seem to be evenly distributed: surgery, cardiovascular, consult, and orthopedic have the most amount of entries.

```r
mtsamples %>% count(medical_specialty)
```

```
## # A tibble: 40 × 2
##    medical_specialty                 n
##    <chr>                         <int>
##  1 " Allergy / Immunology"           7
##  2 " Autopsy"                        8
##  3 " Bariatrics"                    18
##  4 " Cardiovascular / Pulmonary"   372
##  5 " Chiropractic"                  14
##  6 " Consult - History and Phy."   516
##  7 " Cosmetic / Plastic Surgery"    27
##  8 " Dentistry"                     27
##  9 " Dermatology"                   29
## 10 " Diets and Nutritions"          10
## # … with 30 more rows
```

```r
ggplot(data = mtsamples) +
  geom_bar(mapping= aes(y = medical_specialty), color = 'darkgreen',fill = 'green3')
```

![](README_files/figure-html/question1-1.png)<!-- -->

## Question 2
### Tokenize the words in the `transcription` column. Count the number of times each token appears. Visualize the top 20 most frequent words.
Not surprisingly, the majority of the top 20 most frequent words are stop words. The one that stands out is the word *patient*.

```r
mtsamples %>% unnest_tokens(token, transcription) %>% count(token, sort = TRUE) %>% top_n(20, n) %>% ggplot(aes(x = n, y = fct_reorder(token, n ))) + 
  geom_col(color = 'darkgreen', fill = 'green3') +
  labs(title = "Top 20 words", y = "", x = "frequency")
```

![](README_files/figure-html/question2-1.png)<!-- -->

## Question 3

### Redo visualization but remove stopwords as well as numbers.


```r
mtsamples %>%
  unnest_tokens(token, transcription) %>%
  count(token, sort = TRUE) %>% 
  anti_join(stop_words, by = c("token" = "word")) %>% 
  filter(!grepl("^[0-9]+$", x = token)) %>%
  top_n(20, n) %>% 
  ggplot(aes(x = n, y = fct_reorder(token, n ))) + 
    geom_col(color = 'darkgreen', fill = 'green3') +
    labs(title = "Top 20 Words, without Stopwords and Numbers", y = "", x = "frequency")
```

![](README_files/figure-html/question3-1.png)<!-- -->

## Question 4
### Repeat question 2, but this time tokenize into bi-grams. A lot of these contain stop-words, so this is not very informative.

```r
mtsamples %>% unnest_ngrams(ngram, transcription, n = 2) %>% 
  count(ngram, sort = TRUE) %>%
  top_n(20, n) %>%
  ggplot(aes(x = n, y = fct_reorder(ngram, n))) +
  geom_col(color = 'darkgreen', fill = 'green3') +
  labs(title = "Top 20 Bi-grams", y = "", x = "frequency")
```

![](README_files/figure-html/question4-1.png)<!-- -->
### How does the result change if you look at tri-grams?
The top bi-gram *the patient* appears in the top two tri-grams. Some of the other top bi-grams (of the, in the, to the) don't show up as frequently because they are likely being used with a variety of words before/after them. These tri-grams are more unique than the list of bi-grams in containing words that are medical-related.
The frequencies between these two graphs is also very different. The top bi-gram has a frequency of over 20,000 while the top tri-gram has a frequency of approximately 6000.

```r
mtsamples %>% unnest_ngrams(ngram, transcription, n = 3) %>% 
  count(ngram, sort = TRUE) %>%
  top_n(20, n) %>%
  ggplot(aes(x = n, y = fct_reorder(ngram, n))) +
  geom_col(color = 'darkgreen', fill = 'green3') +
  labs(title = "Top 20 Tri-grams", y = "", x = "frequency")
```

![](README_files/figure-html/tri-gram-1.png)<!-- -->

## Question 5
### Using the results you got from questions 4. Pick a word and count the words that appears after and before it.
Picking word *patient*

```r
#mtsamples %>% unnest_ngrams(ngram, transcription, n = 3) %>% 
#  separate(ngram, into = c("word1", "word2", "word3"))
#  count(ngram, sort = TRUE) %>%
#  top_n(20, n) %>%
#  ggplot(aes(x = n, y = fct_reorder(ngram, n))) +
#  geom_col(color = 'darkgreen', fill = 'green3') +
#  labs(title = "Top 20 Tri-grams", y = "", x = "frequency")
```

