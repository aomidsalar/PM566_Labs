---
title: "Week 5 Lab"
author: "Audrey Omidsalar"
date: "9/24/2021"
output:
  html_document:
    toc: yes
    toc_float: yes
    keep_md: yes
  github_document:
  always_allow_html: true
---



## Load Data


```r
if (!file.exists("met_all.gz"))
  download.file(
    url = "https://raw.githubusercontent.com/USCbiostats/data-science-data/master/02_met/met_all.gz",
    destfile = "met_all.gz",
    method   = "libcurl",
    timeout  = 60
    )
met <- data.table::fread("met_all.gz")

stations <- fread("ftp://ftp.ncdc.noaa.gov/pub/data/noaa/isd-history.csv")
```

### Processing / Filtering Data


```r
stations[, USAF := as.integer(USAF)]
```

```
## Warning in eval(jsub, SDenv, parent.frame()): NAs introduced by coercion
```

```r
# Dealing with NAs and 999999
stations[, USAF   := fifelse(USAF == "999999", NA_integer_, USAF)]
stations[, CTRY   := fifelse(CTRY == "", NA_character_, CTRY)]
stations[, STATE  := fifelse(STATE == "", NA_character_, STATE)]

# Selecting the three relevant columns, and keeping unique records
stations <- unique(stations[, list(USAF, CTRY, STATE)])

# Dropping NAs
stations <- stations[!is.na(USAF)]

# Removing duplicates
stations[, n := 1:.N, by = .(USAF)]
stations <- stations[n == 1,][, n := NULL]
```
### Merge the data

```r
##quick way to remove duplicates
stations[, n := 1:.N, by = .(USAF)]
stations <- stations[n == 1,][, n := NULL]
merged <- merge(
  # Data
  x     = met,      
  y     = stations, 
  # List of variables to match
  by.x  = "USAFID",
  by.y  = "USAF", 
  # Which obs to keep?
  all.x = TRUE,      
  all.y = FALSE
  )
```
## Question 1: Representative Station for the US


```r
##There are multiple measurements per station. We have to summarize the data first to characterize each station
merged_station_avg <- merged[, .(
  temp      = mean(temp, na.rm=TRUE),
  wind.sp   = mean(wind.sp, na.rm=TRUE),
  atm.press = mean(atm.press, na.rm=TRUE)), by = c("USAFID", "STATE")
]
```
### Now find the quantiles per variable

```r
merged_station_avg[, temp_med := quantile(temp, probs=0.5, na.rm=TRUE)]
merged_station_avg[, wind.sp_med := quantile(wind.sp, probs=0.5, na.rm=TRUE)]
merged_station_avg[, atm.press_med := quantile(atm.press, probs=0.5, na.rm=TRUE)]


#medians <- merged_station_avg[,.(
#  temp_50      = quantile(temp, probs=0.5, na.rm=TRUE),
#  wind.sp_50   = quantile(wind.sp, probs=0.5, na.rm=TRUE),
#  atm.press_50 = quantile(atm.press, probs=0.5, na.rm=TRUE)
#)]
#medians
```
### Find the station that is closest to these median values using which.min()
The median temperature station is at USAFID 720458.
The median wind speed station is at USAFID 720929.
The median atmospheric pressure station is at USAFID 722238.
The median stations for these three variables do not coincide.

```r
merged_station_avg[which.min(abs(temp - temp_med))]
```

```
##    USAFID STATE     temp  wind.sp atm.press temp_med wind.sp_med atm.press_med
## 1: 720458    KY 23.68173 1.209682       NaN 23.68406    2.461838      1014.691
```

```r
merged_station_avg[which.min(abs(wind.sp - wind.sp_med))]
```

```
##    USAFID STATE     temp  wind.sp atm.press temp_med wind.sp_med atm.press_med
## 1: 720929    WI 17.43278 2.461838       NaN 23.68406    2.461838      1014.691
```

```r
merged_station_avg[which.min(abs(atm.press - atm.press_med))]
```

```
##    USAFID STATE     temp  wind.sp atm.press temp_med wind.sp_med atm.press_med
## 1: 722238    AL 26.13978 1.472656  1014.691 23.68406    2.461838      1014.691
```

```r
#another method
#merged_station_avg[, temp_dist := abs(temp - medians$temp_50)]
#median_temp_station <- merged_station_avg[order(temp_dist)][1]
#median_temp_station
```
## Question 2 Representative Station per State

```r
## add state column by merging
#merged_station_avg <- merge(x = merged_station_avg, y = stations, by.x = "USAFID", by.y = "USAF", all.x = TRUE, all.y=FALSE)
##Compute the median per state
merged_station_avg[,temp_med_state := quantile(temp, probs=.5, na.rm=TRUE), by = "STATE"]
merged_station_avg[,wind.sp_med_state := quantile(wind.sp, probs=.5, na.rm=TRUE), by = "STATE"]
merged_station_avg[,atm.press_med_state := quantile(atm.press, probs=.5, na.rm=TRUE), by = "STATE"]
```


```r
##Compute the Euclidean distance
summary(merged_station_avg$temp)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
##   8.045  20.902  23.684  23.850  26.809  37.625       7
```

```r
summary(merged_station_avg$wind.sp)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
##  0.1842  1.8201  2.4618  2.5784  3.2350 12.0563      14
```

```r
summary(merged_station_avg$atm.press)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
##    1002    1013    1015    1014    1015    1050     693
```

```r
##These datasets have NA's. Remove them to be able to calculate Euclidean distance.
merged_station_avg_noNA <- merged_station_avg[!is.na(temp) & !is.na(wind.sp) & !is.na(atm.press)]
merged_station_avg_noNA[, eudist := sqrt((temp - temp_med_state)^2 + (wind.sp - wind.sp_med_state)^2 + (atm.press - atm.press_med_state)^2)]
merged_station_avg_noNA
```

```
##      USAFID STATE     temp  wind.sp atm.press temp_med wind.sp_med
##   1: 690150    CA 33.18763 3.483560  1010.379 23.68406    2.461838
##   2: 720175    AR 27.97347 1.522316  1014.435 23.68406    2.461838
##   3: 720198    MI 17.68743 3.089919  1014.095 23.68406    2.461838
##   4: 720306    MO 24.07811 3.484249  1014.576 23.68406    2.461838
##   5: 720333    CA 23.59832 2.783191  1012.180 23.68406    2.461838
##  ---                                                              
## 895: 726777    MT 19.15492 4.673878  1014.299 23.68406    2.461838
## 896: 726797    MT 18.78980 2.858586  1014.902 23.68406    2.461838
## 897: 726798    MT 19.47014 4.445783  1014.072 23.68406    2.461838
## 898: 726810    ID 25.03549 3.039794  1011.730 23.68406    2.461838
## 899: 726813    ID 23.47809 2.435372  1012.315 23.68406    2.461838
##      atm.press_med temp_med_state wind.sp_med_state atm.press_med_state
##   1:      1014.691       22.66268          2.565445            1012.557
##   2:      1014.691       26.24296          1.938625            1014.591
##   3:      1014.691       20.51970          2.273423            1014.927
##   4:      1014.691       23.95109          2.453547            1014.522
##   5:      1014.691       22.66268          2.565445            1012.557
##  ---                                                                   
## 895:      1014.691       19.15492          4.151737            1014.185
## 896:      1014.691       19.15492          4.151737            1014.185
## 897:      1014.691       19.15492          4.151737            1014.185
## 898:      1014.691       20.56798          2.568944            1012.855
## 899:      1014.691       20.56798          2.568944            1012.855
##          eudist
##   1: 10.7872106
##   2:  1.7867033
##   3:  3.0627568
##   4:  1.0398834
##   5:  1.0321449
##  ---           
## 895:  0.5343825
## 896:  1.5227689
## 897:  0.4458281
## 898:  4.6308479
## 899:  2.9627420
```

```r
merged_station_avg_noNA[ , .SD[which.min(eudist)], by = STATE]
```

```
##     STATE USAFID     temp  wind.sp atm.press temp_med wind.sp_med atm.press_med
##  1:    CA 722970 22.76040 2.325982  1012.710 23.68406    2.461838      1014.691
##  2:    AR 723407 25.86949 2.208652  1014.575 23.68406    2.461838      1014.691
##  3:    MI 725395 20.44096 2.357275  1015.245 23.68406    2.461838      1014.691
##  4:    MO 723495 24.31621 2.550940  1014.296 23.68406    2.461838      1014.691
##  5:    MD 724057 25.00877 2.033233  1014.497 23.68406    2.461838      1014.691
##  6:    AL 722286 26.35793 1.675828  1014.909 23.68406    2.461838      1014.691
##  7:    OR 725895 18.79793 2.307326  1014.726 23.68406    2.461838      1014.691
##  8:    ID 725867 20.81272 2.702517  1012.802 23.68406    2.461838      1014.691
##  9:    PA 725130 21.69177 1.970192  1015.125 23.68406    2.461838      1014.691
## 10:    KY 724240 23.79463 2.450704  1015.375 23.68406    2.461838      1014.691
## 11:    FL 722106 27.52774 2.711121  1015.322 23.68406    2.461838      1014.691
## 12:    NJ 724090 23.47238 2.148606  1015.095 23.68406    2.461838      1014.691
## 13:    VA 724016 24.29327 1.588105  1014.946 23.68406    2.461838      1014.691
## 14:    TX 722416 29.75394 3.539980  1012.331 23.68406    2.461838      1014.691
## 15:    GA 723160 26.59746 1.684538  1014.985 23.68406    2.461838      1014.691
## 16:    SC 723190 25.73726 2.253408  1015.116 23.68406    2.461838      1014.691
## 17:    OK 723545 27.03555 3.852697  1012.711 23.68406    2.461838      1014.691
## 18:    NV 725805 25.21743 3.101560  1012.461 23.68406    2.461838      1014.691
## 19:    RI 725079 22.27697 2.583469  1014.620 23.68406    2.461838      1014.691
## 20:    LA 722486 28.16413 1.592840  1014.544 23.68406    2.461838      1014.691
## 21:    MS 722358 26.54093 1.747426  1014.722 23.68406    2.461838      1014.691
## 22:    NM 722686 26.00522 4.503611  1012.742 23.68406    2.461838      1014.691
## 23:    AZ 722745 30.31538 3.307632  1010.144 23.68406    2.461838      1014.691
## 24:    CO 724767 21.97732 2.780364  1014.082 23.68406    2.461838      1014.691
## 25:    NC 723174 24.95288 1.744838  1015.350 23.68406    2.461838      1014.691
## 26:    TN 723346 24.59407 1.493531  1015.144 23.68406    2.461838      1014.691
## 27:    DE 724180 24.56026 2.752929  1015.046 23.68406    2.461838      1014.691
## 28:    WV 724176 21.94072 1.649151  1015.982 23.68406    2.461838      1014.691
## 29:    OH 724298 21.79537 2.771958  1015.248 23.68406    2.461838      1014.691
## 30:    IN 725327 22.40044 2.547951  1015.145 23.68406    2.461838      1014.691
## 31:    IL 725440 22.84806 2.566829  1014.760 23.68406    2.461838      1014.691
## 32:    KS 724580 24.01181 3.548029  1013.449 23.68406    2.461838      1014.691
## 33:    UT 725755 24.31031 3.361211  1012.243 23.68406    2.461838      1014.691
## 34:    NY 725194 20.37207 2.444051  1015.327 23.68406    2.461838      1014.691
## 35:    CT 725087 22.57539 2.126514  1014.534 23.68406    2.461838      1014.691
## 36:    MA 725064 21.40933 2.786213  1014.721 23.68406    2.461838      1014.691
## 37:    VT 726115 18.60548 1.101301  1014.985 23.68406    2.461838      1014.691
## 38:    IA 725480 21.43686 2.764312  1014.814 23.68406    2.461838      1014.691
## 39:    NE 725560 21.80411 3.428358  1014.386 23.68406    2.461838      1014.691
## 40:    WY 726650 19.75554 4.243727  1013.527 23.68406    2.461838      1014.691
## 41:    NH 726050 19.86188 1.732752  1014.487 23.68406    2.461838      1014.691
## 42:    ME 726077 18.49969 2.337241  1014.475 23.68406    2.461838      1014.691
## 43:    WI 726452 19.21728 2.411747  1015.180 23.68406    2.461838      1014.691
## 44:    MN 726550 19.11831 2.832794  1015.319 23.68406    2.461838      1014.691
## 45:    SD 726590 19.95928 3.550722  1014.284 23.68406    2.461838      1014.691
## 46:    MT 726798 19.47014 4.445783  1014.072 23.68406    2.461838      1014.691
##     STATE USAFID     temp  wind.sp atm.press temp_med wind.sp_med atm.press_med
##     temp_med_state wind.sp_med_state atm.press_med_state     eudist
##  1:       22.66268          2.565445            1012.557 0.30049511
##  2:       26.24296          1.938625            1014.591 0.46112989
##  3:       20.51970          2.273423            1014.927 0.33875622
##  4:       23.95109          2.453547            1014.522 0.44048404
##  5:       24.89883          1.883499            1014.824 0.37630511
##  6:       26.33664          1.662132            1014.959 0.05608376
##  7:       17.98061          2.011436            1015.269 1.02527449
##  8:       20.56798          2.568944            1012.855 0.28377685
##  9:       21.69177          1.784167            1015.435 0.36234584
## 10:       23.88844          1.895486            1015.245 0.57786362
## 11:       27.57325          2.705069            1015.335 0.04772342
## 12:       23.47238          2.148606            1014.825 0.26971491
## 13:       24.37799          1.653032            1015.158 0.23665335
## 14:       29.75188          3.413737            1012.460 0.18029339
## 15:       26.70404          1.495596            1015.208 0.31157584
## 16:       25.80545          1.696119            1015.281 0.58529642
## 17:       27.14427          3.852697            1012.567 0.18052457
## 18:       24.56293          3.035050            1012.204 0.70623784
## 19:       22.53551          2.583469            1014.728 0.28039594
## 20:       27.87430          1.592840            1014.593 0.29399685
## 21:       26.69258          1.636392            1014.836 0.21966149
## 22:       24.94447          3.776083            1012.525 1.30437627
## 23:       30.32372          3.074359            1010.144 0.23342190
## 24:       21.49638          3.098777            1013.334 0.94422827
## 25:       24.72953          1.627306            1015.420 0.26213187
## 26:       24.88657          1.576035            1015.144 0.30391254
## 27:       24.56026          2.752929            1015.046 0.00000000
## 28:       21.94446          1.633487            1015.762 0.22082482
## 29:       22.02062          2.554397            1015.351 0.32969606
## 30:       22.25059          2.344333            1015.063 0.26577311
## 31:       22.43194          2.237622            1014.760 0.53059335
## 32:       24.21220          3.680613            1013.389 0.24751336
## 33:       24.35182          3.145427            1011.972 0.34923813
## 34:       20.40674          2.304075            1014.887 0.46256996
## 35:       22.36880          2.101801            1014.810 0.34635143
## 36:       21.30662          2.710944            1014.751 0.13084377
## 37:       18.61379          1.408247            1014.792 0.36261055
## 38:       21.33461          2.680875            1014.964 0.19926933
## 39:       21.87354          3.192539            1014.332 0.25159903
## 40:       19.80699          3.873392            1013.157 0.52649035
## 41:       19.55054          1.563826            1014.689 0.40778497
## 42:       18.79016          2.237210            1014.399 0.31653296
## 43:       18.85524          2.053283            1014.893 0.58447881
## 44:       19.63017          2.617071            1015.042 0.62096399
## 45:       20.35662          3.665638            1014.398 0.42910869
## 46:       19.15492          4.151737            1014.185 0.44582815
##     temp_med_state wind.sp_med_state atm.press_med_state     eudist
```
## Question 3
for each state, find median lat and long..find distance using euclidean


## Question 4 - state avg temp already done earlier. then create categorical variable according to temp (low, mid, high). then you can find the summary statistics (bullet points) per categorical variable

```r
#met[, state_temp := mean(temp, na.rm=TRUE), by = "STATE"]
#met[,temp_cat := fifelse(state_temp < 20, "low",
#                         fifelse(state_temp < 25, "mid", "high"))]
#table(met$temp_cat, useNA = "always") ##will also show number of NA's
```
Summary table

```r
#tab <- met[, .(
#  N_entries = .N,
#  N_stations = length(unique(USAFID)),
#  N_states = length(unique(STATE))
#)
#    , by = "temp_cat"]
##make the table look pretty
#knitr::kable(tab)
```

