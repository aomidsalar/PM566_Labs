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

## Processing / Filtering Data


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
### What is the median station in terms of temperature, wind speed, and atmospheric pressure? Look for the three weather stations that best represent continental US using the quantile() function. Do these three coincide?


```r
##There are multiple measurements per station. We have to summarize the data first to characterize each station
merged_station_avg <- merged[, .(
  temp      = mean(temp, na.rm=TRUE),
  wind.sp   = mean(wind.sp, na.rm=TRUE),
  atm.press = mean(atm.press, na.rm=TRUE), lat, lon), by = c("USAFID", "STATE")
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
##    USAFID STATE     temp  wind.sp atm.press    lat     lon temp_med wind.sp_med
## 1: 723010    NC 23.42556 1.641749  1014.729 35.742 -81.382 23.42556    2.309704
##    atm.press_med
## 1:      1014.734
```

```r
merged_station_avg[which.min(abs(wind.sp - wind.sp_med))]
```

```
##    USAFID STATE     temp  wind.sp atm.press    lat     lon temp_med wind.sp_med
## 1: 722143    TX 29.54605 2.309704       NaN 32.579 -96.719 23.42556    2.309704
##    atm.press_med
## 1:      1014.734
```

```r
merged_station_avg[which.min(abs(atm.press - atm.press_med))]
```

```
##    USAFID STATE     temp  wind.sp atm.press    lat     lon temp_med wind.sp_med
## 1: 722316    LA 27.32591 2.014424  1014.734 29.817 -90.017 23.42556    2.309704
##    atm.press_med
## 1:      1014.734
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
##   8.045  20.652  23.426  23.604  26.749  37.625    4408
```

```r
summary(merged_station_avg$wind.sp)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
##   0.184   1.688   2.310   2.459   3.103  12.056   15319
```

```r
summary(merged_station_avg$atm.press)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
##    1002    1013    1015    1014    1015    1050 1456896
```

```r
##These datasets have NA's. Remove them to be able to calculate Euclidean distance.
merged_station_avg_noNA <- merged_station_avg[!is.na(temp) & !is.na(wind.sp) & !is.na(atm.press)]
merged_station_avg_noNA[, eudist := sqrt((temp - temp_med_state)^2 + (wind.sp - wind.sp_med_state)^2 + (atm.press - atm.press_med_state)^2)]
##output data table with euclidean distance annotated
merged_station_avg_noNA
```

```
##         USAFID STATE     temp  wind.sp atm.press    lat      lon temp_med
##      1: 690150    CA 33.18763 3.483560  1010.379 34.300 -116.166 23.42556
##      2: 690150    CA 33.18763 3.483560  1010.379 34.300 -116.166 23.42556
##      3: 690150    CA 33.18763 3.483560  1010.379 34.300 -116.166 23.42556
##      4: 690150    CA 33.18763 3.483560  1010.379 34.300 -116.166 23.42556
##      5: 690150    CA 33.18763 3.483560  1010.379 34.300 -116.166 23.42556
##     ---                                                                  
## 918075: 726813    ID 23.47809 2.435372  1012.315 43.650 -116.633 23.42556
## 918076: 726813    ID 23.47809 2.435372  1012.315 43.650 -116.633 23.42556
## 918077: 726813    ID 23.47809 2.435372  1012.315 43.650 -116.633 23.42556
## 918078: 726813    ID 23.47809 2.435372  1012.315 43.642 -116.636 23.42556
## 918079: 726813    ID 23.47809 2.435372  1012.315 43.642 -116.636 23.42556
##         wind.sp_med atm.press_med temp_med_state wind.sp_med_state
##      1:    2.309704      1014.734       22.32428          2.524027
##      2:    2.309704      1014.734       22.32428          2.524027
##      3:    2.309704      1014.734       22.32428          2.524027
##      4:    2.309704      1014.734       22.32428          2.524027
##      5:    2.309704      1014.734       22.32428          2.524027
##     ---                                                           
## 918075:    2.309704      1014.734       20.16204          2.168066
## 918076:    2.309704      1014.734       20.16204          2.168066
## 918077:    2.309704      1014.734       20.16204          2.168066
## 918078:    2.309704      1014.734       20.16204          2.168066
## 918079:    2.309704      1014.734       20.16204          2.168066
##         atm.press_med_state    eudist
##      1:            1012.708 11.151655
##      2:            1012.708 11.151655
##      3:            1012.708 11.151655
##      4:            1012.708 11.151655
##      5:            1012.708 11.151655
##     ---                              
## 918075:            1012.908  3.379157
## 918076:            1012.908  3.379157
## 918077:            1012.908  3.379157
## 918078:            1012.908  3.379157
## 918079:            1012.908  3.379157
```
This `center_3vars` dataset shows the station with the minimum Euclidean distance (in terms of temperature, wind speed and atmospheric pressure) per state.

```r
center_3vars <- merged_station_avg_noNA[ , .SD[which.min(eudist)], by = STATE]
center_3vars
```

```
##     STATE USAFID     temp  wind.sp atm.press    lat      lon temp_med
##  1:    CA 722977 22.28589 2.364013  1012.653 33.680 -117.866 23.42556
##  2:    AR 723407 25.86949 2.208652  1014.575 35.831  -90.646 23.42556
##  3:    MI 726355 20.43892 1.930327  1014.947 42.126  -86.428 23.42556
##  4:    MO 723495 24.31621 2.550940  1014.296 37.152  -94.495 23.42556
##  5:    MD 724057 25.00877 2.033233  1014.497 39.472  -76.170 23.42556
##  6:    AL 722286 26.35793 1.675828  1014.909 33.212  -87.616 23.42556
##  7:    OR 720365 16.10729 1.468683  1015.957 42.070 -124.290 23.42556
##  8:    ID 722142 20.32324 2.184444  1012.609 44.523 -114.215 23.42556
##  9:    PA 725130 21.69177 1.970192  1015.125 41.333  -75.717 23.42556
## 10:    KY 724243 23.18690 1.458032  1015.642 37.087  -84.077 23.42556
## 11:    FL 722210 27.65745 2.531633  1015.408 30.483  -86.517 23.42556
## 12:    NJ 724075 23.83986 1.949704  1014.825 39.366  -75.078 23.42556
## 13:    VA 724016 24.29327 1.588105  1014.946 38.137  -78.455 23.42556
## 14:    TX 722540 29.98977 3.092698  1012.252 30.300  -97.700 23.42556
## 15:    GA 723160 26.59746 1.684538  1014.985 31.536  -82.507 23.42556
## 16:    SC 723190 25.73726 2.253408  1015.116 34.498  -82.710 23.42556
## 17:    OK 723537 27.05520 3.646514  1012.567 35.852  -97.414 23.42556
## 18:    NV 725830 23.49863 2.966240  1012.679 40.900 -117.800 23.42556
## 19:    RI 725079 22.27697 2.583469  1014.620 41.533  -71.283 23.42556
## 20:    LA 722486 28.16413 1.592840  1014.544 32.516  -92.041 23.42556
## 21:    MS 723306 26.01216 1.827912  1014.830 33.650  -88.450 23.42556
## 22:    NM 723650 26.22686 3.517116  1011.941 35.033 -106.617 23.42556
## 23:    AZ 723740 27.20925 3.677113  1011.656 35.017 -110.733 23.42556
## 24:    CO 724665 20.75472 3.946234  1012.891 39.275 -103.666 23.42556
## 25:    NC 723174 24.95288 1.744838  1015.350 36.047  -79.477 23.42556
## 26:    TN 723346 24.59407 1.493531  1015.144 35.593  -88.917 23.42556
## 27:    DE 724180 24.56026 2.752929  1015.046 39.674  -75.606 23.42556
## 28:    WV 724176 21.94072 1.649151  1015.982 39.643  -79.916 23.42556
## 29:    OH 725254 22.14885 2.330167  1015.117 41.338  -84.429 23.42556
## 30:    IN 725330 21.73189 2.851982  1015.284 40.983  -85.200 23.42556
## 31:    IL 725440 22.84806 2.566829  1014.760 41.450  -90.500 23.42556
## 32:    KS 724580 24.01181 3.548029  1013.449 39.550  -97.650 23.42556
## 33:    UT 725720 26.94480 3.527679  1010.886 40.783 -111.950 23.42556
## 34:    NY 725194 20.37207 2.444051  1015.327 42.643  -77.056 23.42556
## 35:    CT 725087 22.57539 2.126514  1014.534 41.736  -72.651 23.42556
## 36:    MA 725064 21.40933 2.786213  1014.721 41.910  -70.729 23.42556
## 37:    VT 726114 17.46999 1.165761  1014.792 44.534  -72.614 23.42556
## 38:    IA 725480 21.43686 2.764312  1014.814 42.550  -92.400 23.42556
## 39:    NE 725527 22.20987 3.121762  1014.065 41.764  -96.178 23.42556
## 40:    WY 725645 18.39674 4.889281  1012.735 41.317 -105.683 23.42556
## 41:    NH 726050 19.86188 1.732752  1014.487 43.200  -71.500 23.42556
## 42:    ME 726077 18.49969 2.337241  1014.475 44.450  -68.367 23.42556
## 43:    WI 726416 19.12963 1.653207  1014.525 43.212  -90.181 23.42556
## 44:    MN 726550 19.11831 2.832794  1015.319 45.543  -94.051 23.42556
## 45:    SD 726590 19.95928 3.550722  1014.284 45.450  -98.417 23.42556
## 46:    MT 726797 18.78980 2.858586  1014.902 45.788 -111.160 23.42556
##     STATE USAFID     temp  wind.sp atm.press    lat      lon temp_med
##     wind.sp_med atm.press_med temp_med_state wind.sp_med_state
##  1:    2.309704      1014.734       22.32428          2.524027
##  2:    2.309704      1014.734       26.07275          1.861651
##  3:    2.309704      1014.734       20.50971          2.069470
##  4:    2.309704      1014.734       23.99322          2.187797
##  5:    2.309704      1014.734       24.89883          1.600480
##  6:    2.309704      1014.734       26.22064          1.543094
##  7:    2.309704      1014.734       17.16329          1.942080
##  8:    2.309704      1014.734       20.16204          2.168066
##  9:    2.309704      1014.734       21.87141          1.759793
## 10:    2.309704      1014.734       23.68173          1.732910
## 11:    2.309704      1014.734       27.51250          2.531633
## 12:    2.309704      1014.734       23.83986          1.949704
## 13:    2.309704      1014.734       24.30992          1.588105
## 14:    2.309704      1014.734       29.68309          3.150943
## 15:    2.309704      1014.734       26.78250          1.426370
## 16:    2.309704      1014.734       25.73726          1.592177
## 17:    2.309704      1014.734       27.28791          3.646514
## 18:    2.309704      1014.734       23.67835          2.966240
## 19:    2.309704      1014.734       22.53551          2.583469
## 20:    2.309704      1014.734       27.84758          1.459030
## 21:    2.309704      1014.734       26.19508          1.385855
## 22:    2.309704      1014.734       24.94447          3.517116
## 23:    2.309704      1014.734       27.70883          3.023396
## 24:    2.309704      1014.734       20.75472          3.087924
## 25:    2.309704      1014.734       24.51396          1.415669
## 26:    2.309704      1014.734       24.71645          1.513550
## 27:    2.309704      1014.734       24.56026          2.752929
## 28:    2.309704      1014.734       21.94072          1.617823
## 29:    2.309704      1014.734       21.87803          2.368546
## 30:    2.309704      1014.734       21.73189          2.246247
## 31:    2.309704      1014.734       22.33580          2.078367
## 32:    2.309704      1014.734       24.21648          3.679474
## 33:    2.309704      1014.734       26.94480          3.361211
## 34:    2.309704      1014.734       20.37207          2.204099
## 35:    2.309704      1014.734       22.44858          2.077088
## 36:    2.309704      1014.734       21.40933          2.648870
## 37:    2.309704      1014.734       17.87100          1.408247
## 38:    2.309704      1014.734       21.36209          2.633381
## 39:    2.309704      1014.734       21.99129          3.121762
## 40:    2.309704      1014.734       18.43778          3.873392
## 41:    2.309704      1014.734       19.23920          1.556907
## 42:    2.309704      1014.734       18.82098          2.137179
## 43:    2.309704      1014.734       18.71326          1.986436
## 44:    2.309704      1014.734       19.53001          2.357470
## 45:    2.309704      1014.734       19.95928          3.665638
## 46:    2.309704      1014.734       18.78980          3.378081
##     wind.sp_med atm.press_med temp_med_state wind.sp_med_state
##     atm.press_med_state    eudist
##  1:            1012.708 0.1736276
##  2:            1014.591 0.4024542
##  3:            1014.947 0.1561130
##  4:            1014.522 0.5361189
##  5:            1014.824 0.5535881
##  6:            1014.926 0.1917347
##  7:            1015.813 1.1661337
##  8:            1012.908 0.3397841
##  9:            1015.474 0.4460917
## 10:            1015.254 0.6859086
## 11:            1015.335 0.1623900
## 12:            1014.825 0.0000000
## 13:            1015.158 0.2118704
## 14:            1012.456 0.3727459
## 15:            1015.208 0.3884726
## 16:            1015.265 0.6778528
## 17:            1012.711 0.2737236
## 18:            1011.947 0.7535990
## 19:            1014.837 0.3375856
## 20:            1014.593 0.3471884
## 21:            1014.830 0.4784087
## 22:            1012.404 1.3632207
## 23:            1010.144 1.7210317
## 24:            1013.334 0.9658559
## 25:            1015.420 0.5531862
## 26:            1015.144 0.1240055
## 27:            1015.046 0.0000000
## 28:            1015.757 0.2275864
## 29:            1015.351 0.3601454
## 30:            1015.063 0.6447527
## 31:            1014.760 0.7078133
## 32:            1013.389 0.2503865
## 33:            1011.701 0.8310616
## 34:            1014.887 0.5007516
## 35:            1014.810 0.3085369
## 36:            1014.721 0.1373438
## 37:            1014.792 0.4686234
## 38:            1014.957 0.2075646
## 39:            1014.345 0.3549801
## 40:            1013.157 1.1004513
## 41:            1014.689 0.6778409
## 42:            1014.475 0.3784877
## 43:            1014.893 0.6481723
## 44:            1015.042 0.6873763
## 45:            1014.497 0.2428327
## 46:            1014.072 0.9793020
##     atm.press_med_state    eudist
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
<div id="htmlwidget-d13990ddfc2fd39ecd2f" style="width:672px;height:480px;" class="leaflet html-widget"></div>
<script type="application/json" data-for="htmlwidget-d13990ddfc2fd39ecd2f">{"x":{"options":{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}}},"calls":[{"method":"addProviderTiles","args":["OpenStreetMap",null,null,{"errorTileUrl":"","noWrap":false,"detectRetina":false}]},{"method":"addCircles","args":[[37.285,31.15,43.322,34.283,40.477,38.704,35.259,42.6,47.104,32.564,45.097,33.177,40.711,35.584,37.4,41.691,40.217,40.893,43.567,44.778,39,39.173,34.257,35.417,43.064,30.558,37.578,28.228,39.467,40.28,40.033,33.463,38.068,48.39,44.535,33.761,41.384,39.6,38.427,43.767,35.38,42.209,41.597,41.876,39.133,43.205,44.316,45.699,33.68,35.831,42.126,37.152,39.472,33.212,42.07,44.523,41.333,37.087,30.483,39.366,38.137,30.3,31.536,34.498,35.852,40.9,41.533,32.516,33.65,35.033,35.017,39.275,36.047,35.593,39.674,39.643,41.338,40.983,41.45,39.55,40.783,42.643,41.736,41.91,44.534,42.55,41.764,41.317,43.2,44.45,43.212,45.543,45.45,45.788],[-120.512,-97.717,-84.688,-80.567,-88.916,-93.183,-93.093,-123.364,-122.287,-82.985,-94.507,-86.783,-86.375,-79.101,-77.517,-93.566,-76.851,-97.997,-116.24,-89.667,-80.274,-76.684,-111.339,-97.383,-108.458,-92.099,-84.77,-82.156,-106.15,-83.115,-74.353,-105.535,-97.275,-100.024,-72.614,-90.758,-72.506,-116.01,-113.012,-99.318,-86.246,-75.98,-71.412,-71.021,-75.467,-71.503,-69.797,-110.448,-117.866,-90.646,-86.428,-94.495,-76.17,-87.616,-124.29,-114.215,-75.717,-84.077,-86.517,-75.078,-78.455,-97.7,-82.507,-82.71,-97.414,-117.8,-71.283,-92.041,-88.45,-106.617,-110.733,-103.666,-79.477,-88.917,-75.606,-79.916,-84.429,-85.2,-90.5,-97.65,-111.95,-77.056,-72.651,-70.729,-72.614,-92.4,-96.178,-105.683,-71.5,-68.367,-90.181,-94.051,-98.417,-111.16],100,null,null,{"interactive":true,"className":"","stroke":true,"color":["pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange"],"weight":5,"opacity":1,"fill":true,"fillColor":["pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","pink","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange","orange"],"fillOpacity":0.7},null,null,null,{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null,null]}],"limits":{"lat":[28.228,48.39],"lng":[-124.29,-68.367]}},"evals":[],"jsHooks":[]}</script>
```
## Question 4 - Mean of Means


```r
##creating average temperature per state
merged[, state_temp_mean := mean(temp, na.rm=TRUE), by = "STATE"]
##annotating temperature categories
merged[, temp_cat := fifelse(state_temp_mean < 20, "low",
                         fifelse(state_temp_mean < 25, "mid", "high"))]
#table(met$temp_cat, useNA = "always") ##will also show number of NA's
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

