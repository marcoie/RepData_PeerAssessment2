---
output: 
  html_document:
    keep_md: true
---



#Severe Weather Events health and economic impact analysis of NOOA Storm Database   
***

##Synopsis
***


## Data processing
***

### Reading the data
This is based on NOAA Storm Database Analysis coming in the form of a  [*bz2-file*](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2) located on coursera Assignment Page.
To use the data, initially we are going to strip white spaces and modify how to identify NA values by using "" as a valid NA.  


```r
bzfileUrl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2"
download.file(bzfileUrl, destfile = paste0(getwd(), "/StormData.csv.bz2"), method = "curl")
stormdata <- read.table("StormData.csv.bz2", header = TRUE, sep = ",", strip.white = TRUE, stringsAsFactors = FALSE, na.strings = c("NA",""))
str(stormdata)
```

```
## 'data.frame':	902297 obs. of  37 variables:
##  $ STATE__   : num  1 1 1 1 1 1 1 1 1 1 ...
##  $ BGN_DATE  : chr  "4/18/1950 0:00:00" "4/18/1950 0:00:00" "2/20/1951 0:00:00" "6/8/1951 0:00:00" ...
##  $ BGN_TIME  : chr  "0130" "0145" "1600" "0900" ...
##  $ TIME_ZONE : chr  "CST" "CST" "CST" "CST" ...
##  $ COUNTY    : num  97 3 57 89 43 77 9 123 125 57 ...
##  $ COUNTYNAME: chr  "MOBILE" "BALDWIN" "FAYETTE" "MADISON" ...
##  $ STATE     : chr  "AL" "AL" "AL" "AL" ...
##  $ EVTYPE    : chr  "TORNADO" "TORNADO" "TORNADO" "TORNADO" ...
##  $ BGN_RANGE : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ BGN_AZI   : chr  NA NA NA NA ...
##  $ BGN_LOCATI: chr  NA NA NA NA ...
##  $ END_DATE  : chr  NA NA NA NA ...
##  $ END_TIME  : chr  NA NA NA NA ...
##  $ COUNTY_END: num  0 0 0 0 0 0 0 0 0 0 ...
##  $ COUNTYENDN: logi  NA NA NA NA NA NA ...
##  $ END_RANGE : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ END_AZI   : chr  NA NA NA NA ...
##  $ END_LOCATI: chr  NA NA NA NA ...
##  $ LENGTH    : num  14 2 0.1 0 0 1.5 1.5 0 3.3 2.3 ...
##  $ WIDTH     : num  100 150 123 100 150 177 33 33 100 100 ...
##  $ F         : int  3 2 2 2 2 2 2 1 3 3 ...
##  $ MAG       : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ FATALITIES: num  0 0 0 0 0 0 0 0 1 0 ...
##  $ INJURIES  : num  15 0 2 2 2 6 1 0 14 0 ...
##  $ PROPDMG   : num  25 2.5 25 2.5 2.5 2.5 2.5 2.5 25 25 ...
##  $ PROPDMGEXP: chr  "K" "K" "K" "K" ...
##  $ CROPDMG   : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ CROPDMGEXP: chr  NA NA NA NA ...
##  $ WFO       : chr  NA NA NA NA ...
##  $ STATEOFFIC: chr  NA NA NA NA ...
##  $ ZONENAMES : chr  NA NA NA NA ...
##  $ LATITUDE  : num  3040 3042 3340 3458 3412 ...
##  $ LONGITUDE : num  8812 8755 8742 8626 8642 ...
##  $ LATITUDE_E: num  3051 0 0 0 0 ...
##  $ LONGITUDE_: num  8806 0 0 0 0 ...
##  $ REMARKS   : chr  NA NA NA NA ...
##  $ REFNUM    : num  1 2 3 4 5 6 7 8 9 10 ...
```

### Assesing data quality 
As per provided data, STATE variable has not just US state but US territories and maritime zones.  

```r
unique(stormdata$STATE)
```

```
##  [1] "AL" "AZ" "AR" "CA" "CO" "CT" "DE" "DC" "FL" "GA" "HI" "ID" "IL" "IN"
## [15] "IA" "KS" "KY" "LA" "ME" "MD" "MA" "MI" "MN" "MS" "MO" "MT" "NE" "NV"
## [29] "NH" "NJ" "NM" "NY" "NC" "ND" "OH" "OK" "OR" "PA" "RI" "SC" "SD" "TN"
## [43] "TX" "UT" "VT" "VA" "WA" "WV" "WI" "WY" "PR" "AK" "ST" "AS" "GU" "MH"
## [57] "VI" "AM" "LC" "PH" "GM" "PZ" "AN" "LH" "LM" "LE" "LS" "SL" "LO" "PM"
## [71] "PK" "XX"
```

Upon inspection, Event Type is a character variable that has both lowercase and upper case characters, we will make all upper case.

```r
stormdata$EVTYPE <- toupper(stormdata$EVTYPE)
```

Upon content analysis we can observe several 'obvious' typos on event type designation which leads to identify several real and potential dupplication cases, there are *898* different Event Type vallues which a much higher number than 48 mentionened possible values on acompanyng [NOAA Storm Data Documentation from Assignment](https://d396qusza40orc.cloudfront.net/repdata%2Fpeer2_doc%2Fpd01016005curr.pdf) . 


```r
NROW(unique(stormdata$EVTYPE))
```

```
## [1] 898
```

Regarding dates and time zones, although dates are properly formate, time is not so a colon is added, and then the Time Zones need to be set according R lubridate package OlsonNames functions to standardize proper time tracking, for this last Step as per convenience a change to a TZ city will be done.


```r
stormdata$TIME_ZONE <- toupper(stormdata$TIME_ZONE)
unique(stormdata$TIME_ZONE)
```

```
##  [1] "CST" "EST" "PST" "MST" "CDT" "PDT" "EDT" "UNK" "HST" "GMT" "MDT"
## [12] "AST" "ADT" "CSC" "SCT" "ESY" "UTC" "SST" "AKS" "GST"
```

```r
 stormdata <- stormdata %>% 
                mutate(BEGINDATETIME = 
                        mdy_hms(stormdata$BGN_DATE) + 
                        hm(paste0(substr(stormdata$BGN_TIME,1,2),":",substr(stormdata$BGN_TIME,3,4))))
```

To stream Line the monetary impact analysis, we will create PROPDMGUSD and CROPDMGUSD using convention of K meaning 1,000 USD, M meaning 1,000,000 USD and B meaning 1,000,000,000
Several other vars, as show in annex, have too much garbage on them to be properly processed:
* COUNTYNAME has *29601* unique values, which upon ordering show that out of first 1000, 90% are a variation of *"AKZ001xxx"*  

Further 

## Results
***
### Personal Impact


### Economic Impact


### Notes
***
We must have AT LEAST 1 Plot and no more than 3


