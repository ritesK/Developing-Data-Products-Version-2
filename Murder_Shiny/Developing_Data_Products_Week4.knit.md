---
title: "Developing Data Products Week 2 Project"
author: "Mark Davey"
date: "May 26, 2017"
output: html_document
---



#Peer-graded Assignment Week 2 

##Background

The Murder Accountability Project is the most complete database of homicides in the United States currently available. This dataset includes murders from the FBI's Supplementary Homicide Report from 1976 to the present and Freedom of Information Act data on more than 22,000 homicides that were not reported to the Justice Department. This dataset includes the age, race, sex, ethnicity of victims and perpetrators, in addition to the relationship between the victim and perpetrator and weapon used. [Source](https://www.kaggle.com/murderaccountability/homicide-reports)

##Data

The source data for this project are available [here](https://www.kaggle.com/murderaccountability/homicide-reports/downloads/homicide-reports.zip).

The latitude and longitude data for USA 
[here](http://notebook.gaslampmedia.com/wp-content/uploads/2013/08/zip_codes_states.csv)

##Prepare Environment

Clean up old data, locate to working diretory, load the caret package and set the random seed so it is reproduceable. 


```r
rm(list=ls()) #Clean up work area
require("knitr") #We are knitting so lets get the package
opts_knit$set(root.dir = "~/GitHub/data")
library(dplyr) #load required packages
library(leaflet)
library(stringi)
set.seed(3433) #set seed so it's reproducable
```

## Obtain and Load Data

Retrieve the data from the internet if not already local and then load into Data Frames.


```r
#setworking directory
setwd("~/GitHub/data")
#get the data from the remote system and unpack it only if does not exist
if (!file.exists("data")) {
  dir.create("data")
}
##kaggle cannot download the data your ahve to do it manually
#if (!file.exists("./data/homicide-reports.zip")) {
#  fileURL <- "https://www.kaggle.com/murderaccountability/homicide-reports/downloads/homicide-reports.zip"
#  download.file(fileURL,destfile = "./data/homicide-reports.zip",method="libcurl")
#}
if (!file.exists("./data/zip_codes_states.csv")) {
  fileURL <- "http://notebook.gaslampmedia.com/wp-content/uploads/2013/08/zip_codes_states.csv"
  download.file(fileURL,destfile = "./data/zip_codes_states.csv",method="libcurl")
}
#read 
murders <- read.csv(unz("./data/homicide-reports.zip","database.csv"))
locations <- read.csv("./data/zip_codes_states.csv")

dim(murders) #idea of data
```

```
## [1] 638454     24
```

##Clean up the data

This data was large in comparison with our previous examples. Many if the fields are factors or NA so remove all those which are not numeric. Leave "classe" since it is needed as the goal.


```r
colnames(locations) <- stri_trans_totitle(names(locations))
murders$State <- substr(murders$Agency.Code,1,2)
murders$Victim.Count[murders$Victim.Count == 0] <- 1 #victim 0 should be 1
murderTotals <- aggregate(Victim.Count ~ City+State+Year,murders,sum)
murderTotals$County <- murderTotals$City
latlong <- locations %>% regroup(list(quote(City),quote(State))) %>%
summarise(lat = mean(Latitude,na.rm =TRUE),lng = mean(Longitude,na.rm =TRUE))
latlong2 <- locations %>% regroup(list(quote(County),quote(State))) %>%
summarise(lat = mean(Latitude,na.rm =TRUE),lng = mean(Longitude,na.rm =TRUE))
murderLocation <- merge(murderTotals,latlong,by=c("City","State"))
murderLocation2 <- merge(murderTotals,latlong2,by=c("County","State"))
murderLocation$County.x <- NULL
colnames(murderLocation)[5] <- "County"
murderLocation <-unique(rbind(murderLocation,murderLocation2))
murderLocTotals <- aggregate(Victim.Count ~ City+State+lat+lng,murderLocation,sum)
murderLocTotals$hover <- with(murderLocTotals, paste(City,State, '<br>', "Count:", Victim.Count))
# Make state borders white
```

## Plot Murders


```r
murderLocTotals %>%  
  leaflet() %>%
  addTiles() %>%
  addMarkers(label=murderLocation$City) %>%
  addCircles(weight = 1, radius = sqrt(murderLocation$Victim.Count) * 500)
```

```
## Assuming 'lng' and 'lat' are longitude and latitude, respectively
## Assuming 'lng' and 'lat' are longitude and latitude, respectively
```

<!--html_preserve--><div id="htmlwidget-fe6802d6aac90f376079" style="width:672px;height:480px;" class="leaflet html-widget"></div>


## Summary

The data has problems since some cities are missing and some locations seem a little off. But clearly large population centers have more murders.