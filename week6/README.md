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
patient <- mtsamples %>% unnest_ngrams(ngram, transcription, n = 3) %>% 
  separate(ngram, into = c("word1", "word2", "word3"), sep = " ") %>%
  select(word1, word2, word3) %>%
  filter(word2 == "patient")
##Frequency of words that appear before the word 'patient'
patient %>% count(word1, sort = TRUE) %>% knitr::kable()
```



|word1            |     n|
|:----------------|-----:|
|the              | 20294|
|this             |   463|
|history          |   101|
|a                |    67|
|and              |    47|
|procedure        |    32|
|female           |    26|
|with             |    25|
|use              |    24|
|old              |    23|
|sample           |    23|
|male             |    22|
|new              |    19|
|general          |    16|
|illness          |    16|
|plan             |    16|
|indications      |    15|
|to               |    15|
|allergies        |    14|
|your             |    13|
|correct          |    11|
|detail           |    11|
|of               |    11|
|course           |    10|
|normal           |    10|
|exam             |     9|
|for              |     9|
|lbs              |     9|
|in               |     8|
|instructions     |     8|
|minutes          |     8|
|recommend        |     8|
|systems          |     8|
|day              |     7|
|digits           |     7|
|s                |     7|
|subjective       |     7|
|that             |     7|
|1                |     6|
|2                |     6|
|doctor           |     6|
|established      |     6|
|eyes             |     6|
|hyperthyroidism  |     6|
|injury           |     6|
|my               |     6|
|none             |     6|
|obtained         |     6|
|pain             |     6|
|route            |     6|
|social           |     6|
|technique        |     6|
|activity         |     5|
|therapy          |     5|
|weeks            |     5|
|4                |     4|
|appropriate      |     4|
|bilaterally      |     4|
|breast           |     4|
|cardiovascular   |     4|
|constitutional   |     4|
|dermatologic     |     4|
|each             |     4|
|ed               |     4|
|findings         |     4|
|further          |     4|
|gastrointestinal |     4|
|genitourinary    |     4|
|goals            |     4|
|ill              |     4|
|infection        |     4|
|infusion         |     4|
|integumentary    |     4|
|medications      |     4|
|neurological     |     4|
|observation      |     4|
|per              |     4|
|pleasant         |     4|
|preprocedure     |     4|
|psychiatric      |     4|
|respiratory      |     4|
|room             |     4|
|seizures         |     4|
|source           |     4|
|strips           |     4|
|treatment        |     4|
|uncertainty      |     4|
|yes              |     4|
|young            |     4|
|after            |     3|
|as               |     3|
|but              |     3|
|cardio           |     3|
|children         |     3|
|cuff             |     3|
|depression       |     3|
|dermatology      |     3|
|diabetes         |     3|
|discharge        |     3|
|discouraged      |     3|
|encouraged       |     3|
|endocrine        |     3|
|gi               |     3|
|gu               |     3|
|health           |     3|
|heent            |     3|
|impotence        |     3|
|laceration       |     3|
|monitor          |     3|
|oriented         |     3|
|p.r.n            |     3|
|pm               |     3|
|pregnancy        |     3|
|presentation     |     3|
|resp             |     3|
|site             |     3|
|symptoms         |     3|
|take             |     3|
|team             |     3|
|today            |     3|
|total            |     3|
|units            |     3|
|weight           |     3|
|when             |     3|
|which            |     3|
|05               |     2|
|3                |     2|
|58               |     2|
|8                |     2|
|98               |     2|
|98.6             |     2|
|above            |     2|
|activities       |     2|
|adenoma          |     2|
|admission        |     2|
|again            |     2|
|agent            |     2|
|ago              |     2|
|alcohol          |     2|
|appearance       |     2|
|areas            |     2|
|arthralgias      |     2|
|asplenic         |     2|
|atelectasis      |     2|
|back             |     2|
|bilateral        |     2|
|breath           |     2|
|brings           |     2|
|case             |     2|
|clearance        |     2|
|collapse         |     2|
|complaint        |     2|
|complaints       |     2|
|condition        |     2|
|contents         |     2|
|context          |     2|
|contraception    |     2|
|coughing         |     2|
|data             |     2|
|decompression    |     2|
|delightful       |     2|
|difficulty       |     2|
|disease          |     2|
|dl               |     2|
|domain           |     2|
|down             |     2|
|eclampsia        |     2|
|effusion         |     2|
|employed         |     2|
|enlarging        |     2|
|examination      |     2|
|exercise         |     2|
|extensive        |     2|
|extremities      |     2|
|factors          |     2|
|fat              |     2|
|fibrillation     |     2|
|fist             |     2|
|from             |     2|
|gestation        |     2|
|gout             |     2|
|habits           |     2|
|hand             |     2|
|healthy          |     2|
|heavy            |     2|
|hemiparesis      |     2|
|hemostasis       |     2|
|herniation       |     2|
|hpi              |     2|
|hypertension     |     2|
|indication       |     2|
|laboratory       |     2|
|loss             |     2|
|maintenance      |     2|
|mammography      |     2|
|manner           |     2|
|married          |     2|
|minute           |     2|
|months           |     2|
|needs            |     2|
|nice             |     2|
|no               |     2|
|note             |     2|
|noted            |     2|
|on               |     2|
|patient          |     2|
|pneumonia        |     2|
|position         |     2|
|preoperatively   |     2|
|primary          |     2|
|proper           |     2|
|protocol         |     2|
|rash             |     2|
|rectum           |     2|
|recurrence       |     2|
|regularly        |     2|
|removed          |     2|
|rest             |     2|
|sampling         |     2|
|scarred          |     2|
|sedentary        |     2|
|see              |     2|
|severe           |     2|
|size             |     2|
|skills           |     2|
|stating          |     2|
|stitch           |     2|
|suctioned        |     2|
|surgery          |     2|
|syncope          |     2|
|syndrome         |     2|
|taken            |     2|
|then             |     2|
|time             |     2|
|tolerable        |     2|
|train            |     2|
|tube             |     2|
|unremarkable     |     2|
|vagina           |     2|
|visit            |     2|
|vomiting         |     2|
|weakness         |     2|
|week             |     2|
|weekend          |     2|
|where            |     2|
|withdrawn        |     2|
|workstation      |     2|
|yyyy             |     2|
|01               |     1|
|aid              |     1|
|arrival          |     1|
|breasts          |     1|
|cough            |     1|
|daytime          |     1|
|directly         |     1|
|dysfunction      |     1|
|guidance         |     1|
|improved         |     1|
|intercourse      |     1|
|light            |     1|
|movement         |     1|
|parents          |     1|
|performance      |     1|
|put              |     1|
|reports          |     1|
|restrictions     |     1|
|term             |     1|
|vol              |     1|

```r
##Frequency of words that appear after the word 'patient'
patient %>% count(word3, sort = TRUE) %>% knitr::kable()
```



|word3            |    n|
|:----------------|----:|
|was              | 6291|
|is               | 3332|
|has              | 1417|
|tolerated        |  994|
|had              |  886|
|will             |  616|
|denies           |  552|
|and              |  377|
|states           |  363|
|does             |  334|
|also             |  301|
|in               |  246|
|did              |  232|
|to               |  215|
|the              |  200|
|underwent        |  180|
|received         |  160|
|reports          |  155|
|with             |  141|
|understood       |  113|
|presents         |   91|
|should           |   83|
|lives            |   81|
|who              |   79|
|on               |   77|
|presented        |   72|
|admits           |   69|
|appears          |   68|
|for              |   67|
|including        |   67|
|would            |   65|
|denied           |   62|
|then             |   61|
|reported         |   58|
|at               |   57|
|remained         |   56|
|as               |   52|
|went             |   52|
|understands      |   51|
|subsequently     |   49|
|of               |   47|
|stated           |   46|
|she              |   44|
|may              |   40|
|continued        |   38|
|returned         |   38|
|agreed           |   36|
|he               |   36|
|a                |   35|
|continues        |   35|
|after            |   34|
|came             |   34|
|comes            |   32|
|under            |   32|
|appeared         |   31|
|i                |   31|
|back             |   30|
|currently        |   30|
|desires          |   30|
|left             |   29|
|began            |   28|
|experienced      |   28|
|felt             |   28|
|identification   |   28|
|that             |   28|
|could            |   27|
|developed        |   27|
|started          |   27|
|today            |   27|
|complains        |   25|
|describes        |   25|
|this             |   25|
|being            |   24|
|informed         |   24|
|takes            |   24|
|discharged       |   23|
|notes            |   23|
|said             |   23|
|but              |   22|
|can              |   22|
|complained       |   22|
|initially        |   22|
|needs            |   22|
|noted            |   22|
|from             |   21|
|became           |   20|
|says             |   20|
|about            |   19|
|apparently       |   19|
|brought          |   18|
|cannot           |   18|
|gave             |   18|
|regarding        |   18|
|relates          |   18|
|still            |   18|
|recently         |   17|
|smokes           |   17|
|however          |   16|
|returns          |   16|
|seems            |   16|
|it               |   15|
|shows            |   15|
|ambulates        |   14|
|be               |   14|
|by               |   14|
|decided          |   14|
|exercised        |   14|
|herself          |   14|
|needed           |   14|
|procedure        |   14|
|all              |   13|
|moves            |   13|
|risks            |   13|
|up               |   13|
|elected          |   12|
|having           |   12|
|instructions     |   12|
|wished           |   12|
|woke             |   12|
|arrived          |   11|
|completed        |   11|
|gives            |   11|
|history          |   11|
|include          |   11|
|physical         |   11|
|return           |   11|
|suffered         |   11|
|already          |   10|
|becomes          |   10|
|demonstrates     |   10|
|feels            |   10|
|follow           |   10|
|himself          |   10|
|home             |   10|
|improved         |   10|
|lying            |   10|
|now              |   10|
|recovered        |   10|
|refuses          |   10|
|requested        |   10|
|we               |   10|
|2                |    9|
|an               |    9|
|because          |    9|
|care             |    9|
|her              |    9|
|includes         |    9|
|later            |    9|
|prepped          |    9|
|provided         |    9|
|taken            |    9|
|took             |    9|
|undergo          |    9|
|walks            |    9|
|weighs           |    9|
|abc              |    8|
|again            |    8|
|along            |    8|
|brings           |    8|
|clinically       |    8|
|consent          |    8|
|discussing       |    8|
|indicates        |    8|
|made             |    8|
|met              |    8|
|passed           |    8|
|position         |    8|
|previously       |    8|
|quit             |    8|
|rates            |    8|
|requests         |    8|
|scored           |    8|
|transferred      |    8|
|wanted           |    8|
|according        |    7|
|actually         |    7|
|died             |    7|
|family           |    7|
|included         |    7|
|into             |    7|
|might            |    7|
|once             |    7|
|operative        |    7|
|placed           |    7|
|positioning      |    7|
|referred         |    7|
|right            |    7|
|speaks           |    7|
|spent            |    7|
|upon             |    7|
|used             |    7|
|when             |    7|
|which            |    7|
|works            |    7|
|admitted         |    6|
|daughter         |    6|
|dear             |    6|
|declined         |    6|
|described        |    6|
|exhibits         |    6|
|his              |    6|
|movement         |    6|
|not              |    6|
|opted            |    6|
|or               |    6|
|oriented         |    6|
|plans            |    6|
|questionnaire    |    6|
|refused          |    6|
|relationship     |    6|
|remains          |    6|
|resides          |    6|
|severe           |    6|
|signed           |    6|
|spends           |    6|
|supine           |    6|
|there            |    6|
|unable           |    6|
|3                |    5|
|although         |    5|
|are              |    5|
|asked            |    5|
|been             |    5|
|continue         |    5|
|eats             |    5|
|ended            |    5|
|expressed        |    5|
|first            |    5|
|have             |    5|
|indicated        |    5|
|just             |    5|
|last             |    5|
|maintained       |    5|
|never            |    5|
|performs         |    5|
|plan             |    5|
|prior            |    5|
|since            |    5|
|sustained        |    5|
|talking          |    5|
|tends            |    5|
|tolerating       |    5|
|verbalized       |    5|
|vomited          |    5|
|wants            |    5|
|were             |    5|
|accepted         |    4|
|advises          |    4|
|assessment       |    4|
|attended         |    4|
|believes         |    4|
|clearly          |    4|
|closely          |    4|
|consented        |    4|
|deteriorated     |    4|
|dr               |    4|
|drinks           |    4|
|during           |    4|
|either           |    4|
|extremity        |    4|
|fell             |    4|
|followed         |    4|
|goal             |    4|
|got              |    4|
|identified       |    4|
|immediately      |    4|
|increase         |    4|
|instructed       |    4|
|looks            |    4|
|makes            |    4|
|no               |    4|
|nothing          |    4|
|obtained         |    4|
|only             |    4|
|otherwise        |    4|
|over             |    4|
|overnight        |    4|
|participated     |    4|
|positioned       |    4|
|probably         |    4|
|progressed       |    4|
|pushed           |    4|
|put              |    4|
|regained         |    4|
|required         |    4|
|reviewing        |    4|
|ruled            |    4|
|seemed           |    4|
|sees             |    4|
|stopped          |    4|
|tells            |    4|
|therefore        |    4|
|told             |    4|
|unfortunately    |    4|
|voiced           |    4|
|well             |    4|
|wishes           |    4|
|without          |    4|
|13               |    3|
|2.5              |    3|
|achieved         |    3|
|attempted        |    3|
|based            |    3|
|calcium          |    3|
|called           |    3|
|claimed          |    3|
|compression      |    3|
|consisting       |    3|
|consulted        |    3|
|cries            |    3|
|deficits         |    3|
|delivered        |    3|
|deny             |    3|
|desired          |    3|
|dynamics         |    3|
|elects           |    3|
|enjoys           |    3|
|estimates        |    3|
|eventually       |    3|
|explained        |    3|
|followup         |    3|
|forgot           |    3|
|given            |    3|
|here             |    3|
|impression       |    3|
|information      |    3|
|laid             |    3|
|maintains        |    3|
|meds             |    3|
|need             |    3|
|originally       |    3|
|other            |    3|
|out              |    3|
|post             |    3|
|postoperatively  |    3|
|presenting       |    3|
|proceeded        |    3|
|profile          |    3|
|progressively    |    3|
|ran              |    3|
|ranks            |    3|
|receive          |    3|
|reduce           |    3|
|sat              |    3|
|say              |    3|
|scheduled        |    3|
|seated           |    3|
|significant      |    3|
|sitting          |    3|
|smoked           |    3|
|spoke            |    3|
|state            |    3|
|status           |    3|
|subjectively     |    3|
|taking           |    3|
|tests            |    3|
|though           |    3|
|tried            |    3|
|turns            |    3|
|tylenol          |    3|
|use              |    3|
|weight           |    3|
|while            |    3|
|within           |    3|
|work             |    3|
|worked           |    3|
|37               |    2|
|4                |    2|
|600              |    2|
|72               |    2|
|acutely          |    2|
|additionally     |    2|
|af               |    2|
|afterward        |    2|
|aggressive       |    2|
|agrees           |    2|
|alert            |    2|
|allergies        |    2|
|alternatives     |    2|
|ambulated        |    2|
|ample            |    2|
|answered         |    2|
|attempts         |    2|
|attends          |    2|
|awake            |    2|
|awoke            |    2|
|axial            |    2|
|bathes           |    2|
|begin            |    2|
|beginning        |    2|
|best             |    2|
|between          |    2|
|biological       |    2|
|bradycardic      |    2|
|breast           |    2|
|breathing        |    2|
|call             |    2|
|catheterizing    |    2|
|certainly        |    2|
|chart            |    2|
|claims           |    2|
|condition        |    2|
|confirmed        |    2|
|consumed         |    2|
|continually      |    2|
|converted        |    2|
|correctly        |    2|
|cpt              |    2|
|declines         |    2|
|deemed           |    2|
|despite          |    2|
|diagnoses        |    2|
|discomfort       |    2|
|discontinue      |    2|
|dorsal           |    2|
|draped           |    2|
|eat              |    2|
|emphatically     |    2|
|endorsed         |    2|
|endorses         |    2|
|enters           |    2|
|enthusiastically |    2|
|especially       |    2|
|evaluation       |    2|
|even             |    2|
|ever             |    2|
|exhibited        |    2|
|experimented     |    2|
|expresses        |    2|
|extubated        |    2|
|eye              |    2|
|failed           |    2|
|finished         |    2|
|follows          |    2|
|found            |    2|
|get              |    2|
|giving           |    2|
|goals            |    2|
|goes             |    2|
|going            |    2|
|heal             |    2|
|healed           |    2|
|hemodynamically  |    2|
|horizontal       |    2|
|hospital         |    2|
|hospitalized     |    2|
|identifies       |    2|
|if               |    2|
|incapable        |    2|
|incidentally     |    2|
|ingested         |    2|
|interestingly    |    2|
|intraoperatively |    2|
|intubated        |    2|
|iv               |    2|
|jerked           |    2|
|knee             |    2|
|knew             |    2|
|knows            |    2|
|laboratory       |    2|
|likely           |    2|
|lost             |    2|
|mainly           |    2|
|maximized        |    2|
|medically        |    2|
|medication       |    2|
|mini             |    2|
|missing          |    2|
|most             |    2|
|motivation       |    2|
|moved            |    2|
|mrs              |    2|
|nature           |    2|
|neglected        |    2|
|noncompliant     |    2|
|noticed          |    2|
|npo              |    2|
|nursing          |    2|
|off              |    2|
|opting           |    2|
|paced            |    2|
|past             |    2|
|patient          |    2|
|per              |    2|
|performed        |    2|
|physician        |    2|
|planned          |    2|
|please           |    2|
|possesses        |    2|
|postprocedure    |    2|
|preoperative     |    2|
|preoperatively   |    2|
|pressure         |    2|
|properly         |    2|
|psychologic      |    2|
|pursuing         |    2|
|rapidly          |    2|
|re               |    2|
|reached          |    2|
|recalls          |    2|
|receives         |    2|
|recheck          |    2|
|recollection     |    2|
|refusing         |    2|
|related          |    2|
|replies          |    2|
|reportedly       |    2|
|responded        |    2|
|restricted       |    2|
|retained         |    2|
|retains          |    2|
|retired          |    2|
|revealed         |    2|
|reviewed         |    2|
|saw              |    2|
|secondary        |    2|
|see              |    2|
|seeks            |    2|
|seen             |    2|
|semi             |    2|
|sent             |    2|
|set              |    2|
|showed           |    2|
|sign             |    2|
|simply           |    2|
|sincerely        |    2|
|slipped          |    2|
|snores           |    2|
|so               |    2|
|sought           |    2|
|stabilize        |    2|
|stands           |    2|
|stating          |    2|
|stood            |    2|
|stop             |    2|
|stopping         |    2|
|stressing        |    2|
|suffers          |    2|
|supportive       |    2|
|supposedly       |    2|
|terminated       |    2|
|through          |    2|
|together         |    2|
|tpa              |    2|
|try              |    2|
|tympanic         |    2|
|ultimately       |    2|
|undergoes        |    2|
|unlike           |    2|
|upper            |    2|
|urinates         |    2|
|using            |    2|
|usually          |    2|
|utilized         |    2|
|verbalizes       |    2|
|very             |    2|
|via              |    2|
|visit            |    2|
|vitamin          |    2|
|voices           |    2|
|weighed          |    2|
|where            |    2|
|whether          |    2|
|whom             |    2|
|wife's           |    2|
|wore             |    2|
|wound            |    2|
|xxx              |    2|
|abcd             |    1|
|ate              |    1|
|cut              |    1|
|excluding        |    1|
|expired          |    1|
|landed           |    1|
|leaving          |    1|
|lived            |    1|
|managed          |    1|
|medical          |    1|
|primarily        |    1|
|recalled         |    1|
|routinely        |    1|
|safety           |    1|
|slept            |    1|
|special          |    1|
|stayed           |    1|
|they             |    1|
|transforaminally |    1|
|walked           |    1|

