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

#stations <- fread("ftp://ftp.ncdc.noaa.gov/pub/data/noaa/isd-history.csv")
stations <- fread("ftp://ftp.ncdc.noaa.gov/pub/data/noaa/isd-history.csv")
```

## Processing / Filtering Data


```r
stations[, USAF := as.integer(USAF)]
```

```
## Warning in eval(jsub, SDenv, parent.frame()): NAs introduced by coercion
```

```r
stations[, USAF   := fifelse(USAF == 999999, NA_integer_, USAF)]
stations[, CTRY   := fifelse(CTRY == "", NA_character_, CTRY)]
stations[, STATE  := fifelse(STATE == "", NA_character_, STATE)]

# Selecting the three relevant columns, and keeping unique records
stations <- unique(stations[, list(USAF, CTRY, STATE)])
stations <- stations[!is.na(USAF)]
##easy way to remove duplicates
stations[, n := 1:.N, by = .(USAF)]
stations <- stations[n == 1,][, n := NULL]
```
### Merge the data

```r
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
### What is the median station in terms of temperature, wind speed, and atmospheric pressure? Look for the three weather stations that best represent continental US using the quantile() function. Do these three coincide?


```r
##There are multiple measurements per station. We have to summarize the data first to characterize each station
merged_station_avg <- merged[,.(
  temp = mean(temp,na.rm=TRUE),
  wind.sp = mean(wind.sp, na.rm=TRUE),
  atm.press = mean(atm.press, na.rm=TRUE), 
  lon = mean(lon, na.rm = TRUE),
  lat = mean(lat, na.rm = TRUE)
), by= c("USAFID", "STATE")]
```
### Now find the quantiles per variable

```r
merged_station_avg[, temp_med := quantile(temp, probs=0.5, na.rm=TRUE)]
merged_station_avg[, wind.sp_med := quantile(wind.sp, probs=0.5, na.rm=TRUE)]
merged_station_avg[, atm.press_med := quantile(atm.press, probs=0.5, na.rm=TRUE)]


medians <- merged_station_avg[,.(
  temp_50      = quantile(temp, probs=0.5, na.rm=TRUE),
  wind.sp_50   = quantile(wind.sp, probs=0.5, na.rm=TRUE),
  atm.press_50 = quantile(atm.press, probs=0.5, na.rm=TRUE)
)]
medians
```

```
##     temp_50 wind.sp_50 atm.press_50
## 1: 23.68406   2.461838     1014.691
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
##    USAFID STATE     temp  wind.sp atm.press     lon    lat temp_med wind.sp_med
## 1: 720458    KY 23.68173 1.209682       NaN -82.637 37.751 23.68406    2.461838
##    atm.press_med
## 1:      1014.691
```

```r
merged_station_avg[which.min(abs(wind.sp - wind.sp_med))]
```

```
##    USAFID STATE     temp  wind.sp atm.press     lon    lat temp_med wind.sp_med
## 1: 720929    WI 17.43278 2.461838       NaN -91.981 45.506 23.68406    2.461838
##    atm.press_med
## 1:      1014.691
```

```r
merged_station_avg[which.min(abs(atm.press - atm.press_med))]
```

```
##    USAFID STATE     temp  wind.sp atm.press       lon     lat temp_med
## 1: 722238    AL 26.13978 1.472656  1014.691 -85.66667 31.3499 23.68406
##    wind.sp_med atm.press_med
## 1:    2.461838      1014.691
```

```r
#another method
#merged_station_avg[, temp_dist := abs(temp - medians$temp_50)]
#median_temp_station <- merged_station_avg[order(temp_dist)][1]
#median_temp_station
```
## Question 2 Representative Station per State
### Just like the previous question, you are asked to identify what is the most representative, the median, station per state. This time, instead of looking at one variable at a time, look at the euclidean distance. If multiple stations show in the median, select the one located at the lowest latitude.

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
##Calculate Euclidean Distance
merged_station_avg_noNA[, eudist := sqrt((temp - temp_med_state)^2 + (wind.sp - wind.sp_med_state)^2 + (atm.press - atm.press_med_state)^2)]
```
This `center_3vars` dataset shows the station with the minimum Euclidean distance (in terms of temperature, wind speed and atmospheric pressure) per state.

```r
center_3vars <- merged_station_avg_noNA[ , .SD[which.min(eudist)], by = STATE]
center_3vars
```

```
##     STATE USAFID     temp  wind.sp atm.press        lon      lat temp_med
##  1:    CA 722970 22.76040 2.325982  1012.710 -118.14652 33.81264 23.68406
##  2:    AR 723407 25.86949 2.208652  1014.575  -90.64600 35.83100 23.68406
##  3:    MI 725395 20.44096 2.357275  1015.245  -84.46697 42.26697 23.68406
##  4:    MO 723495 24.31621 2.550940  1014.296  -94.49501 37.15200 23.68406
##  5:    MD 724057 25.00877 2.033233  1014.497  -76.16996 39.47174 23.68406
##  6:    AL 722286 26.35793 1.675828  1014.909  -87.61600 33.21200 23.68406
##  7:    OR 725895 18.79793 2.307326  1014.726 -121.72405 42.14705 23.68406
##  8:    ID 725867 20.81272 2.702517  1012.802 -113.76605 42.54201 23.68406
##  9:    PA 725130 21.69177 1.970192  1015.125  -75.72500 41.33380 23.68406
## 10:    KY 724240 23.79463 2.450704  1015.375  -85.96723 37.90032 23.68406
## 11:    FL 722106 27.52774 2.711121  1015.322  -81.86101 26.58501 23.68406
## 12:    NJ 724090 23.47238 2.148606  1015.095  -74.35016 40.03300 23.68406
## 13:    VA 724016 24.29327 1.588105  1014.946  -78.45499 38.13701 23.68406
## 14:    TX 722416 29.75394 3.539980  1012.331  -98.04599 29.70899 23.68406
## 15:    GA 723160 26.59746 1.684538  1014.985  -82.50700 31.53600 23.68406
## 16:    SC 723190 25.73726 2.253408  1015.116  -82.71000 34.49800 23.68406
## 17:    OK 723545 27.03555 3.852697  1012.711  -97.08896 36.16199 23.68406
## 18:    NV 725805 25.21743 3.101560  1012.461 -118.56898 40.06799 23.68406
## 19:    RI 725079 22.27697 2.583469  1014.620  -71.28300 41.53300 23.68406
## 20:    LA 722486 28.16413 1.592840  1014.544  -92.04098 32.51596 23.68406
## 21:    MS 722358 26.54093 1.747426  1014.722  -90.47100 31.18298 23.68406
## 22:    NM 722686 26.00522 4.503611  1012.742 -103.31565 34.38358 23.68406
## 23:    AZ 722745 30.31538 3.307632  1010.144 -110.88300 32.16695 23.68406
## 24:    CO 724767 21.97732 2.780364  1014.082 -108.62600 37.30699 23.68406
## 25:    NC 723174 24.95288 1.744838  1015.350  -79.47700 36.04700 23.68406
## 26:    TN 723346 24.59407 1.493531  1015.144  -88.91700 35.59302 23.68406
## 27:    DE 724180 24.56026 2.752929  1015.046  -75.60600 39.67400 23.68406
## 28:    WV 724176 21.94072 1.649151  1015.982  -79.91600 39.64300 23.68406
## 29:    OH 724298 21.79537 2.771958  1015.248  -84.02700 40.70800 23.68406
## 30:    IN 725327 22.40044 2.547951  1015.145  -87.00600 41.45300 23.68406
## 31:    IL 725440 22.84806 2.566829  1014.760  -90.52032 41.46325 23.68406
## 32:    KS 724580 24.01181 3.548029  1013.449  -97.65090 39.55090 23.68406
## 33:    UT 725755 24.31031 3.361211  1012.243 -111.96637 41.11737 23.68406
## 34:    NY 725194 20.37207 2.444051  1015.327  -77.05599 42.64299 23.68406
## 35:    CT 725087 22.57539 2.126514  1014.534  -72.65098 41.73601 23.68406
## 36:    MA 725064 21.40933 2.786213  1014.721  -70.72900 41.91000 23.68406
## 37:    VT 726115 18.60548 1.101301  1014.985  -72.51800 43.34400 23.68406
## 38:    IA 725480 21.43686 2.764312  1014.814  -92.40088 42.55358 23.68406
## 39:    NE 725560 21.80411 3.428358  1014.386  -97.43479 41.98568 23.68406
## 40:    WY 726650 19.75554 4.243727  1013.527 -105.54099 44.33905 23.68406
## 41:    NH 726050 19.86188 1.732752  1014.487  -71.50245 43.20409 23.68406
## 42:    ME 726077 18.49969 2.337241  1014.475  -68.36677 44.45000 23.68406
## 43:    WI 726452 19.21728 2.411747  1015.180  -89.83701 44.35900 23.68406
## 44:    MN 726550 19.11831 2.832794  1015.319  -94.05102 45.54301 23.68406
## 45:    SD 726590 19.95928 3.550722  1014.284  -98.41344 45.44377 23.68406
## 46:    MT 726798 19.47014 4.445783  1014.072 -110.44004 45.69800 23.68406
##     STATE USAFID     temp  wind.sp atm.press        lon      lat temp_med
##     wind.sp_med atm.press_med temp_med_state wind.sp_med_state
##  1:    2.461838      1014.691       22.66268          2.565445
##  2:    2.461838      1014.691       26.24296          1.938625
##  3:    2.461838      1014.691       20.51970          2.273423
##  4:    2.461838      1014.691       23.95109          2.453547
##  5:    2.461838      1014.691       24.89883          1.883499
##  6:    2.461838      1014.691       26.33664          1.662132
##  7:    2.461838      1014.691       17.98061          2.011436
##  8:    2.461838      1014.691       20.56798          2.568944
##  9:    2.461838      1014.691       21.69177          1.784167
## 10:    2.461838      1014.691       23.88844          1.895486
## 11:    2.461838      1014.691       27.57325          2.705069
## 12:    2.461838      1014.691       23.47238          2.148606
## 13:    2.461838      1014.691       24.37799          1.653032
## 14:    2.461838      1014.691       29.75188          3.413737
## 15:    2.461838      1014.691       26.70404          1.495596
## 16:    2.461838      1014.691       25.80545          1.696119
## 17:    2.461838      1014.691       27.14427          3.852697
## 18:    2.461838      1014.691       24.56293          3.035050
## 19:    2.461838      1014.691       22.53551          2.583469
## 20:    2.461838      1014.691       27.87430          1.592840
## 21:    2.461838      1014.691       26.69258          1.636392
## 22:    2.461838      1014.691       24.94447          3.776083
## 23:    2.461838      1014.691       30.32372          3.074359
## 24:    2.461838      1014.691       21.49638          3.098777
## 25:    2.461838      1014.691       24.72953          1.627306
## 26:    2.461838      1014.691       24.88657          1.576035
## 27:    2.461838      1014.691       24.56026          2.752929
## 28:    2.461838      1014.691       21.94446          1.633487
## 29:    2.461838      1014.691       22.02062          2.554397
## 30:    2.461838      1014.691       22.25059          2.344333
## 31:    2.461838      1014.691       22.43194          2.237622
## 32:    2.461838      1014.691       24.21220          3.680613
## 33:    2.461838      1014.691       24.35182          3.145427
## 34:    2.461838      1014.691       20.40674          2.304075
## 35:    2.461838      1014.691       22.36880          2.101801
## 36:    2.461838      1014.691       21.30662          2.710944
## 37:    2.461838      1014.691       18.61379          1.408247
## 38:    2.461838      1014.691       21.33461          2.680875
## 39:    2.461838      1014.691       21.87354          3.192539
## 40:    2.461838      1014.691       19.80699          3.873392
## 41:    2.461838      1014.691       19.55054          1.563826
## 42:    2.461838      1014.691       18.79016          2.237210
## 43:    2.461838      1014.691       18.85524          2.053283
## 44:    2.461838      1014.691       19.63017          2.617071
## 45:    2.461838      1014.691       20.35662          3.665638
## 46:    2.461838      1014.691       19.15492          4.151737
##     wind.sp_med atm.press_med temp_med_state wind.sp_med_state
##     atm.press_med_state     eudist
##  1:            1012.557 0.30049511
##  2:            1014.591 0.46112989
##  3:            1014.927 0.33875622
##  4:            1014.522 0.44048404
##  5:            1014.824 0.37630511
##  6:            1014.959 0.05608376
##  7:            1015.269 1.02527449
##  8:            1012.855 0.28377685
##  9:            1015.435 0.36234584
## 10:            1015.245 0.57786362
## 11:            1015.335 0.04772342
## 12:            1014.825 0.26971491
## 13:            1015.158 0.23665335
## 14:            1012.460 0.18029339
## 15:            1015.208 0.31157584
## 16:            1015.281 0.58529642
## 17:            1012.567 0.18052457
## 18:            1012.204 0.70623784
## 19:            1014.728 0.28039594
## 20:            1014.593 0.29399685
## 21:            1014.836 0.21966149
## 22:            1012.525 1.30437627
## 23:            1010.144 0.23342190
## 24:            1013.334 0.94422827
## 25:            1015.420 0.26213187
## 26:            1015.144 0.30391254
## 27:            1015.046 0.00000000
## 28:            1015.762 0.22082482
## 29:            1015.351 0.32969606
## 30:            1015.063 0.26577311
## 31:            1014.760 0.53059335
## 32:            1013.389 0.24751336
## 33:            1011.972 0.34923813
## 34:            1014.887 0.46256996
## 35:            1014.810 0.34635143
## 36:            1014.751 0.13084377
## 37:            1014.792 0.36261055
## 38:            1014.964 0.19926933
## 39:            1014.332 0.25159903
## 40:            1013.157 0.52649035
## 41:            1014.689 0.40778497
## 42:            1014.399 0.31653296
## 43:            1014.893 0.58447881
## 44:            1015.042 0.62096399
## 45:            1014.398 0.42910869
## 46:            1014.185 0.44582815
##     atm.press_med_state     eudist
```
## Question 3: In the Middle?
### For each state, identify what is the station that is closest to the mid-point of the state. Combining these with the stations you identified in the previous question, use leaflet() to visualize all ~100 points in the same figure, applying different colors for those identified in this question.

```r
##Finding median latitude and longitude per state
merged[, lat_median := quantile(lat, probs=.5, na.rm=TRUE), by = "STATE"]
merged[, lon_median := quantile(lon, probs=.5, na.rm=TRUE), by = "STATE"]
##Finding the Euclidean distances between latitude and longitude
merged[, eudist_latlon := sqrt((lat - lat_median)^2 + (lon - lon_median)^2)]
##Outputting rows with minimum Euclidean distance per state
center_state <- merged[ , .SD[which.min(eudist_latlon)], by = STATE]
center_state
```

```
##     STATE USAFID  WBAN year month day hour min    lat      lon elev wind.dir
##  1:    CA 724815 23257 2019     8   1    0  53 37.285 -120.512   48      310
##  2:    TX 722570  3933 2019     8   1    0  58 31.150  -97.717  282       90
##  3:    MI 725405 54816 2019     8   1    0  15 43.322  -84.688  230       90
##  4:    SC 720603   195 2019     8   1    0  15 34.283  -80.567   92       NA
##  5:    IL 724397 54831 2019     8   9   11  56 40.477  -88.916  265      350
##  6:    MO 724453  3994 2019     8   1    0  53 38.704  -93.183  276      350
##  7:    AR 723429 53920 2019     8  13    0  53 35.259  -93.093  123       NA
##  8:    OR 725975 24235 2019     8   1    0  56 42.600 -123.364 1171       NA
##  9:    WA 720388   469 2019     8   1    0  15 47.104 -122.287  164       NA
## 10:    GA 722217 63881 2019     8   1    0  15 32.564  -82.985   94       NA
## 11:    MN 726583  4941 2019     8   1    0  55 45.097  -94.507  347      120
## 12:    AL 722300 53864 2019     8  13    0  53 33.177  -86.783  179      220
## 13:    IN 720961   336 2019     8   1    0  15 40.711  -86.375  225      340
## 14:    NC 722201  3723 2019     8   9   11  35 35.584  -79.101   75       NA
## 15:    VA 720498   153 2019     8   1    0  56 37.400  -77.517   72       90
## 16:    IA 725466  4938 2019     8   1    0  15 41.691  -93.566  277       NA
## 17:    PA 725118 14751 2019     8   1    0  56 40.217  -76.851  106      240
## 18:    NE 725513  4901 2019     8   1    0  15 40.893  -97.997  550      110
## 19:    ID 726810 24131 2019     8   1    0  53 43.567 -116.240  874      330
## 20:    WI 726465 94890 2019     8   1    0  55 44.778  -89.667  389       NA
## 21:    WV 720328 63832 2019     8   1    0  15 39.000  -80.274  498      100
## 22:    MD 724060 93721 2019     8   1    0  54 39.173  -76.684   47       NA
## 23:    AZ 723745   374 2019     8   1    0  15 34.257 -111.339 1572       NA
## 24:    OK 723540 13919 2019     8   1    0  56 35.417  -97.383  393      160
## 25:    WY 726720 24061 2019     8   1    0  53 43.064 -108.458 1684      150
## 26:    LA 720468   466 2019     8   1    0  15 30.558  -92.099   23      160
## 27:    KY 720448   144 2019     8   1    0  15 37.578  -84.770  312       NA
## 28:    FL 721042   486 2019     8   1    0  15 28.228  -82.156   27      140
## 29:    CO 722061  3038 2019     8   1    0  14 39.467 -106.150 3680      100
## 30:    OH 720928   315 2019     8   1    0  15 40.280  -83.115  288       10
## 31:    NJ 724090 14780 2019     8   5   22   0 40.033  -74.353   31       90
## 32:    NM 722683 93083 2019     8   1    8  55 33.463 -105.535 2077       NA
## 33:    KS 724509 53939 2019     8   1    6  56 38.068  -97.275  467      120
## 34:    ND 720867   293 2019     8   1    0  15 48.390 -100.024  472      190
## 35:    VT 726114 54771 2019     8   1   18  54 44.535  -72.614  223       NA
## 36:    MS 725023   474 2019     8   9   11  35 33.761  -90.758   43       NA
## 37:    CT 720545   169 2019     8   1    0  15 41.384  -72.506  127       NA
## 38:    NV 724770  3170 2019     8   1   12  53 39.600 -116.010 1812      200
## 39:    UT 724750 23176 2019     8   1    0  52 38.427 -113.012 1536       NA
## 40:    SD 726530 94943 2019     8   1    0  15 43.767  -99.318  517      130
## 41:    TN 721031   348 2019     8   1    0  15 35.380  -86.246  330      180
## 42:    NY 725150  4725 2019     8  13    1  53 42.209  -75.980  499      210
## 43:    RI 725074 54752 2019     8   1    0  50 41.597  -71.412    5       NA
## 44:    MA 725068 54777 2019     8   1    0  52 41.876  -71.021   13       NA
## 45:    DE 724088 13707 2019     8   1    0  18 39.133  -75.467    9       70
## 46:    NH 726050 14745 2019     8   1    0  51 43.205  -71.503  105      160
## 47:    ME 726185 14605 2019     8   1    0  53 44.316  -69.797  110      290
## 48:    MT 726798 24150 2019     8  13    0  53 45.699 -110.448 1420      280
##     STATE USAFID  WBAN year month day hour min    lat      lon elev wind.dir
##     wind.dir.qc wind.type.code wind.sp wind.sp.qc ceiling.ht ceiling.ht.qc
##  1:           5              N     4.1          5      22000             5
##  2:           1              N     4.1          1      22000             1
##  3:           5              N     1.5          5      22000             5
##  4:           9              C     0.0          5      22000             5
##  5:           1              N     2.6          1      22000             1
##  6:           5              N     5.1          5       3048             5
##  7:           9              C     0.0          1      22000             1
##  8:           9              V     1.5          5      22000             5
##  9:           9              C     0.0          5      22000             5
## 10:           9              C     0.0          5      22000             5
## 11:           1              N     3.1          1      22000             5
## 12:           1              N     1.5          1       2134             1
## 13:           5              N     3.6          5      22000             5
## 14:           9              C     0.0          1      22000             1
## 15:           1              N     1.5          1      22000             5
## 16:           9              N      NA          9      22000             5
## 17:           5              N     3.1          5      22000             5
## 18:           5              N     5.7          5      22000             5
## 19:           5              N     4.1          5      22000             5
## 20:           9              C     0.0          1      22000             1
## 21:           5              N     1.5          5         NA             9
## 22:           9              C     0.0          5       4877             5
## 23:           9              C     0.0          5       3658             5
## 24:           5              N     3.6          5      22000             5
## 25:           5              N     3.1          5      22000             5
## 26:           5              N     2.6          5      22000             5
## 27:           9              C     0.0          5      22000             5
## 28:           5              N     3.6          5       3353             5
## 29:           5              N     5.7          5       2896             5
## 30:           5              N     3.1          5      22000             5
## 31:           1              N     3.6          1      22000             1
## 32:           9              C     0.0          1         NA             9
## 33:           5              N     6.2          5         NA             9
## 34:           5              N     3.6          5      22000             5
## 35:           9              V     3.1          1      22000             1
## 36:           9              C     0.0          1      22000             5
## 37:           9              C     0.0          5      22000             5
## 38:           1              N     1.5          1         NA             9
## 39:           9              C     0.0          1      22000             1
## 40:           5              N     5.1          5      22000             5
## 41:           5              N     2.6          5       1524             5
## 42:           1              N     3.1          1      22000             5
## 43:           9              C     0.0          5       4572             5
## 44:           9              C     0.0          5      22000             5
## 45:           5              N     5.1          5       2134             5
## 46:           5              N     1.5          5      22000             5
## 47:           5              N     2.1          5       1676             5
## 48:           1              N     6.2          1      22000             1
##     wind.dir.qc wind.type.code wind.sp wind.sp.qc ceiling.ht ceiling.ht.qc
##     ceiling.ht.method sky.cond vis.dist vis.dist.qc vis.var vis.var.qc temp
##  1:                 9        N    16093           5       N          5 35.0
##  2:                 9        N    16093           1       9          9 34.5
##  3:                 9        N    16093           5       N          5 22.7
##  4:                 9        N    16093           5       N          5 29.0
##  5:                 9        N    16093           1       9          9 17.8
##  6:                 M        N    16093           5       N          5 21.1
##  7:                 9        N    16093           1       9          9 30.0
##  8:                 9        N    16093           5       N          5 25.0
##  9:                 9        N    16093           5       N          5 29.0
## 10:                 9        N    16093           5       N          5 29.0
## 11:                 9        N    16093           1       9          9 25.0
## 12:                 9        N    16093           1       9          9 32.2
## 13:                 9        N    16093           5       N          5 22.0
## 14:                 9        N    16093           1       9          9 20.0
## 15:                 9        N    16093           1       9          9 23.3
## 16:                 9        N    16093           5       N          5 19.0
## 17:                 9        N    16093           5       N          5 26.7
## 18:                 9        N    16093           5       N          5 26.6
## 19:                 9        N    16093           5       N          5 36.1
## 20:                 9        N    16093           1       9          9 21.0
## 21:                 9        N    16093           5       N          5 22.0
## 22:                 M        N    16093           5       N          5 26.7
## 23:                 M        N    16093           5       N          5 26.0
## 24:                 9        N    16093           5       N          5 32.3
## 25:                 9        N    16093           5       N          5 27.2
## 26:                 9        N    16093           5       N          5 27.8
## 27:                 9        N    16093           5       N          5 26.0
## 28:                 M        N    11265           5       N          5 22.7
## 29:                 M        N    16093           5       N          5 15.0
## 30:                 9        N    16093           5       N          5 25.0
## 31:                 9        N       NA           9       9          9 26.1
## 32:                 9        N    16093           1       9          9 20.0
## 33:                 9        N    16093           5       N          5 25.0
## 34:                 9        N    16093           5       N          5 29.0
## 35:                 9        N    16093           1       9          9 24.4
## 36:                 9        N    16093           1       9          9 24.8
## 37:                 9        N    11265           5       N          5 21.0
## 38:                 9        N       NA           9       9          9 12.2
## 39:                 9        N    16093           1       9          9 24.4
## 40:                 9        N    16093           5       N          5 26.2
## 41:                 M        N    16093           5       N          5 26.0
## 42:                 9        N    16093           1       9          9 21.7
## 43:                 M        N    16093           5       N          5 25.0
## 44:                 9        N    16093           5       N          5 23.3
## 45:                 M        N    16093           5       N          5 26.0
## 46:                 9        N     8047           5       N          5 21.1
## 47:                 M        N     8047           5       N          5 22.2
## 48:                 9        N    16093           1       9          9 23.3
##     ceiling.ht.method sky.cond vis.dist vis.dist.qc vis.var vis.var.qc temp
##     temp.qc dew.point dew.point.qc atm.press atm.press.qc        rh CTRY
##  1:       5       7.2            5    1010.5            5  17.85877   US
##  2:       1      18.6            1        NA            9  38.96081   US
##  3:       5       9.1            5        NA            9  42.02597   US
##  4:       5      22.0            5        NA            9  65.95870   US
##  5:       1      12.8            1    1015.3            1  72.70817   US
##  6:       5      20.6            5    1021.9            5  96.98682   US
##  7:       1      25.0            1    1009.2            1  74.59676   US
##  8:       5      10.6            5    1014.1            5  40.41716   US
##  9:       5      10.0            5        NA            9  30.60164   US
## 10:       5      20.0            5        NA            9  58.32577   US
## 11:       1      16.0            1        NA            9  57.46078   US
## 12:       1      22.8            1    1011.5            1  57.57832   US
## 13:       5      15.0            5        NA            9  64.62394   US
## 14:       1      19.9            1        NA            9  99.38627   US
## 15:       1      17.2            1    1018.4            1  68.68188   US
## 16:       5      18.0            5        NA            9  93.96867   US
## 17:       5      20.6            5    1016.6            5  69.27841   US
## 18:       5      19.1            5        NA            9  63.50739   US
## 19:       5       1.1            5    1007.8            5  10.87160   US
## 20:       1      12.0            1        NA            9  56.56040   US
## 21:       5      18.0            5        NA            9  78.14902   US
## 22:       5      20.0            5    1016.3            5  66.76005   US
## 23:       5      14.0            5        NA            9  47.59617   US
## 24:       5      19.5            5        NA            9  46.72222   US
## 25:       5       6.1            5    1014.0            5  26.09109   US
## 26:       5      23.2            5        NA            9  76.09515   US
## 27:       5      20.0            5        NA            9  69.58648   US
## 28:       5      22.1            5        NA            9  96.43240   US
## 29:       5       5.0            5        NA            9  51.46430   US
## 30:       5      18.0            5        NA            9  65.20835   US
## 31:       1      21.7            1    1013.6            1  76.78892   US
## 32:       1       7.0            1        NA            9  43.04672   US
## 33:       5      21.1            5    1014.4            5  79.02548   US
## 34:       5      21.0            5        NA            9  62.03959   US
## 35:       1      10.0            1    1020.4            1  40.26115   US
## 36:       1      24.8            1        NA            9 100.00000   US
## 37:       5      21.0            5        NA            9 100.00000   US
## 38:       1       6.7            1    1014.3            1  69.34826   US
## 39:       1      12.8            1    1015.0            1  48.45233   US
## 40:       5      24.4            5        NA            9  89.85960   US
## 41:       5      22.0            5        NA            9  78.67081   US
## 42:       1      15.6            1    1011.7            1  68.39318   US
## 43:       5      18.0            5        NA            9  65.20835   US
## 44:       5      20.0            5    1016.3            5  81.78746   US
## 45:       5      24.0            5    1016.5            5  88.77636   US
## 46:       5      20.6            5    1015.8            5  96.98682   US
## 47:       5      16.7            5    1016.3            5  71.13935   US
## 48:       1       3.3            1    1015.7            1  27.14411   US
##     temp.qc dew.point dew.point.qc atm.press atm.press.qc        rh CTRY
##     lat_median lon_median eudist_latlon
##  1:     36.780   -120.448    0.50903929
##  2:     31.178    -97.691    0.03820995
##  3:     43.067    -84.688    0.25500000
##  4:     34.181    -80.634    0.12203688
##  5:     40.520    -88.751    0.17051100
##  6:     38.583    -93.183    0.12100000
##  7:     35.333    -92.767    0.33429328
##  8:     42.600   -123.364    0.00000000
##  9:     47.104   -122.416    0.12900000
## 10:     32.564    -83.270    0.28500000
## 11:     45.097    -94.382    0.12500000
## 12:     32.915    -86.557    0.34600578
## 13:     40.711    -86.296    0.07900000
## 14:     35.633    -79.101    0.04900000
## 15:     37.321    -77.558    0.08900562
## 16:     41.717    -93.566    0.02600000
## 17:     40.435    -76.922    0.22927058
## 18:     41.189    -98.054    0.30143822
## 19:     43.650   -116.240    0.08300000
## 20:     44.614    -89.774    0.19581879
## 21:     38.885    -80.400    0.17059015
## 22:     38.981    -76.684    0.19200000
## 23:     34.257   -111.733    0.39400000
## 24:     35.534    -97.350    0.12156480
## 25:     42.796   -108.389    0.27673995
## 26:     30.521    -91.983    0.12175796
## 27:     37.591    -84.672    0.09885848
## 28:     28.290    -81.876    0.28678215
## 29:     39.217   -105.861    0.38212694
## 30:     40.333    -83.078    0.06463745
## 31:     40.277    -74.417    0.25225384
## 32:     34.067   -105.990    0.75620169
## 33:     38.329    -97.430    0.30355560
## 34:     48.301    -99.621    0.41271055
## 35:     44.567    -72.562    0.06105735
## 36:     33.433    -90.078    0.75497285
## 37:     41.384    -72.682    0.17600000
## 38:     39.300   -116.891    0.93067771
## 39:     38.427   -113.012    0.00000000
## 40:     44.045    -99.318    0.27800000
## 41:     35.593    -86.246    0.21300000
## 42:     42.241    -75.412    0.56890069
## 43:     41.597    -71.433    0.02100000
## 44:     42.098    -70.918    0.24473046
## 45:     39.133    -75.467    0.00000000
## 46:     43.278    -71.503    0.07300000
## 47:     44.316    -69.667    0.13000000
## 48:     45.788   -110.440    0.08935883
##     lat_median lon_median eudist_latlon
```
Combine the two datasets

```r
center_state[, label := "Center of the state"]
center_3vars[, label := "Center of wind speed, temperature, and atmospheric pressure"]
centers <- rbind(center_state, center_3vars, fill=TRUE)
```
Create map. The median center of state stations are shown in pink, and the stations corresponding to the median of the three variables (wind speed, atmospheric pressure and temperature) are in orange.

```r
leaflet(centers) %>%
  addProviderTiles("OpenStreetMap") %>%
  addCircles(lng = ~lon, lat = ~lat, color=~ifelse(label=="Center of the state",'pink','orange'), opacity=1,fillOpacity=0.7, radius=100)
```

```{=html}
<div id="htmlwidget-c4ed99d13a1fc96e0d22" style="width:672px;height:480px;" class="leaflet html-widget"></div>
<script type="application/json" data-for="htmlwidget-c4ed99d13a1fc96e0d22">{"x":{"options":{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}}},"calls":[{"method":"addProviderTiles","args":["OpenStreetMap",null,null,{"errorTileUrl":"","noWrap":false,"detectRetina":false}]},{"method":"addCircles","args":[[37.285,31.15,43.322,34.283,40.477,38.704,35.259,42.6,47.104,32.564,45.097,33.177,40.711,35.584,37.4,41.691,40.217,40.893,43.567,44.778,39,39.173,34.257,35.417,43.064,30.558,37.578,28.228,39.467,40.28,40.033,33.463,38.068,48.39,44.535,33.761,41.384,39.6,38.427,43.767,35.38,42.209,41.597,41.876,39.133,43.205,44.316,45.699,33.8126445959104,35.831000967118,42.2669724409449,37.152,39.4717428571429,33.212,42.1470528169014,42.5420088495575,41.3338032388664,37.9003206568712,26.5850051546392,40.033,38.137011684518,29.7089922705314,31.536,34.4979967776584,36.1619871031746,40.0679923664122,41.5329991281604,32.5159599542334,31.1829822852081,34.3835781584582,32.1669504405286,37.3069902080783,36.047,35.5930237288136,39.6740047984645,39.643,40.708,41.4530010638298,41.4632502351834,39.5509035769829,41.1173656050955,42.642987628866,41.7360101010101,41.9099972527472,43.344,42.5535839285714,41.9856836734694,44.3390462962963,43.2040901033973,44.45,44.3590049261084,45.5430087241003,45.443765323993,45.6980046136102],[-120.512,-97.717,-84.688,-80.567,-88.916,-93.183,-93.093,-123.364,-122.287,-82.985,-94.507,-86.783,-86.375,-79.101,-77.517,-93.566,-76.851,-97.997,-116.24,-89.667,-80.274,-76.684,-111.339,-97.383,-108.458,-92.099,-84.77,-82.156,-106.15,-83.115,-74.353,-105.535,-97.275,-100.024,-72.614,-90.758,-72.506,-116.01,-113.012,-99.318,-86.246,-75.98,-71.412,-71.021,-75.467,-71.503,-69.797,-110.448,-118.146523855891,-90.646,-84.466968503937,-94.4950114942529,-76.1699571428571,-87.616,-121.724052816901,-113.766053097345,-75.7249967611336,-85.9672290406223,-81.8610051546392,-74.3501562130177,-78.454988315482,-98.0459922705314,-82.507,-82.7099989258861,-97.0889613095238,-118.568984732824,-71.2829991281604,-92.04097597254,-90.4710035429584,-103.315653104925,-110.883,-108.626004895961,-79.477,-88.9169966101695,-75.6060009596929,-79.916,-84.027,-87.0060010638298,-90.5203170272813,-97.6509035769829,-111.966365605096,-77.055993814433,-72.6509797979798,-70.729,-72.5179990974729,-92.4008803571429,-97.4347891156463,-105.540990740741,-71.5024542097489,-68.3667746192893,-89.8370098522168,-94.0510196292257,-98.413442206655,-110.440038062284],100,null,null,{"interactive":true,"className":"","stroke":true,"color":["pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange"],"weight":5,"opacity":1,"fill":true,"fillColor":["pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange"],"fillOpacity":0.7},null,null,null,{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null,null]}],"limits":{"lat":[26.5850051546392,48.39],"lng":[-123.364,-68.3667746192893]}},"evals":[],"jsHooks":[]}</script>
```
## Question 4 - Mean of Means


```r
##creating average temperature per state
merged[, state_temp_mean := mean(temp, na.rm=TRUE), by = "STATE"]
##annotating temperature categories
merged[, temp_cat := fifelse(state_temp_mean < 20, "low",
                         fifelse(state_temp_mean < 25, "mid", "high"))]
table(merged$temp_cat, useNA = "always") ##will show number of NA's
```

```
## 
##    high     low     mid    <NA> 
##  811126  430794 1135423       0
```
Summary table

```r
##creating table with assigned variables
tab <- merged[, .(
  N_entries      = .N,
  N_na           = sum(is.na(temp)),
  N_stations     = length(unique(USAFID)),
  N_states       = length(unique(STATE)),
  mean_temp      = mean(temp, na.rm=TRUE),
  mean_wind.sp   = mean(wind.sp, na.rm=TRUE),
  mean_atm.press = mean(atm.press, na.rm=TRUE)
)
    , by = "temp_cat"]
##make the table look pretty
knitr::kable(tab)
```



|temp_cat | N_entries|  N_na| N_stations| N_states| mean_temp| mean_wind.sp| mean_atm.press|
|:--------|---------:|-----:|----------:|--------:|---------:|------------:|--------------:|
|mid      |   1135423| 29252|        781|       25|  22.39909|     2.352712|       1014.383|
|high     |    811126| 23468|        555|       12|  27.75066|     2.514644|       1013.738|
|low      |    430794|  7369|        259|       11|  18.96446|     2.637410|       1014.366|

