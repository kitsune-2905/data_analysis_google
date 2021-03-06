# data_analysis_google
data analyst project for google data analyst certificate written in R


---
title: "Case Study Coursera Google Data Analyst"
author: "Stefan Fuchs"
date: "8 12 2021"
output: html_document
---



### Project Description

For the Google Data Analysts course on [coursera](https://www.coursera.org) I work with a sample data set about bicycle rentals to find differences between annual members and casual riders.



### Data Description
As a first step I downloaded the data from source which was provided. The data I will use extends over a period of 12 months and are provided in csv format and are downloadable in zip format in individual files for each month. The data starts with April of 2019 and ends with March 2020. The data is provided via online download and presents as internal data.
The data is licensed under [DivvY](https://www.divvybikes.com/data-license-agreement), follow the link to find out more.



### Tools for Analyzis

In this case study there are multiple tasks been given, and to the overall extend the data set presents, I decided to use RStudio to log all my changes and also create the needed visualizations. 



### Business Task

In the case study the question asked is : **How do annual members and casual riders use Cyclistic bikes diﬀerently?**

The business task is to store the data within a data frame and inspect it for any errors, duplicates, empty fields and clear those and document every step involved. Then I will inspect the data for insights and include different metrics for analyzes and reporting. Derived from the insights I will create some visualizations of the data for stakeholders and 



### Creating environment

First I unzipped all the files and put them into my project folder under csv. I then had a first look onto the data and found geolocations (longitude and latitude) within the data set. So I decided to use the tidyverse package and the geosphere package. Do to me using a linux system I installed the packages directly via the command line.

```{bash eval=FALSE, include=FALSE}
sudo apt install r-cran-tidyvers r-cran-geosphere r-cran-lubridate
```

Including libraries to RStudio
```{r}
library(tidyverse)
library(geosphere)
library(lubridate)
```



### Data Preparation and Processing

As base for this step I used the resource provided by [Divvy Exercise R Script](https://docs.google.com/document/d/1TTj5KNKf4BWvEORGm10oNbpwTRk1hamsWJGj6qRWpuI/edit).

change the working director for all code chunks
```{r, setup, include=FALSE}
knitr::opts_knit$set(root.dir = "~/2112-case_study_bicycle/csv")
```


### Collect Data

import all data to seperate data frames for inspection
```{r}
q2_2019 <- read_csv("Divvy_Trips_2019_Q2.csv")
q3_2019 <- read_csv("Divvy_Trips_2019_Q3.csv")
q4_2019 <- read_csv("Divvy_Trips_2019_Q4.csv")
q1_2020 <- read_csv("Divvy_Trips_2020_Q1.csv")
```


### Wrangle Data And Combine Into A Single File

the colomn names changed in the file q1_2019 so we will change them to fit
```{r include=FALSE}
(q4_2019 <- rename(q4_2019
                   ,ride_id = trip_id
                   ,rideable_type = bikeid 
                   ,started_at = start_time  
                   ,ended_at = end_time  
                   ,start_station_name = from_station_name 
                   ,start_station_id = from_station_id 
                   ,end_station_name = to_station_name 
                   ,end_station_id = to_station_id 
                   ,member_casual = usertype))
(q3_2019 <- rename(q3_2019
                   ,ride_id = trip_id
                   ,rideable_type = bikeid 
                   ,started_at = start_time  
                   ,ended_at = end_time  
                   ,start_station_name = from_station_name 
                   ,start_station_id = from_station_id 
                   ,end_station_name = to_station_name 
                   ,end_station_id = to_station_id 
                   ,member_casual = usertype))
(q2_2019 <- rename(q2_2019
                   ,ride_id = "01 - Rental Details Rental ID"
                   ,rideable_type = "01 - Rental Details Bike ID" 
                   ,started_at = "01 - Rental Details Local Start Time"  
                   ,ended_at = "01 - Rental Details Local End Time"  
                   ,start_station_name = "03 - Rental Start Station Name" 
                   ,start_station_id = "03 - Rental Start Station ID"
                   ,end_station_name = "02 - Rental End Station Name" 
                   ,end_station_id = "02 - Rental End Station ID"
                   ,member_casual = "User Type"))
```

inspection of data frames and incongruencies
```{r include=FALSE}
str(q1_2020)
str(q4_2019)
str(q3_2019)
str(q2_2019)
```

convert ride_id and ridable_type to character so that they can stack correctly
```{r}
q4_2019 <-  mutate(q4_2019, ride_id = as.character(ride_id)
                   ,rideable_type = as.character(rideable_type)) 
q3_2019 <-  mutate(q3_2019, ride_id = as.character(ride_id)
                   ,rideable_type = as.character(rideable_type)) 
q2_2019 <-  mutate(q2_2019, ride_id = as.character(ride_id)
                   ,rideable_type = as.character(rideable_type)) 
```

Stack individual quarter's data frames into one big data frame
```{r}
all_trips <- bind_rows(q2_2019, q3_2019, q4_2019, q1_2020)
```

Remove lat, long, birthyear, and gender fields as this data was dropped beginning in 2020
```{r}
all_trips <- all_trips %>%  
  select(-c(start_lat, start_lng, end_lat, end_lng, birthyear, gender, "01 - Rental Details Duration In Seconds Uncapped", "05 - Member Details Member Birthday Year", "Member Gender", "tripduration"))
```


### Clean Up and Add Data to Prepare for Analysis

some initial inspection off the new dataset (all_trips)
```{r}
colnames(all_trips)  #List of column names
nrow(all_trips)  #How many rows are in data frame?
dim(all_trips)  #Dimensions of the data frame?
head(all_trips)  #See the first 6 rows of data frame.  Also tail(all_trips)
str(all_trips)  #See list of columns and data types (numeric, character, etc)
summary(all_trips)  #Statistical summary of data. Mainly for numerics
```

There are a few problems we will need to fix:
(1) In the "member_casual" column, there are two names for members ("member" and "Subscriber") and two names for casual riders ("Customer" and "casual"). We will need to consolidate that from four to two labels.
(2) The data can only be aggregated at the ride-level, which is too granular. We will want to add some additional columns of data -- such as day, month, year -- that provide additional opportunities to aggregate the data.
(3) We will want to add a calculated field for length of ride since the 2020Q1 data did not have the "tripduration" column. We will add "ride_length" to the entire dataframe for consistency.
(4) There are some rides where tripduration shows up as negative, including several hundred rides where Divvy took bikes out of circulation for Quality Control reasons. We will want to delete these rides.

In the "member_casual" column, replace "Subscriber" with "member" and "Customer" with "casual"
Before 2020, Divvy used different labels for these two types of riders ... we will want to make our dataframe consistent with their current nomenclature
N.B.: "Level" is a special property of a column that is retained even if a subset does not contain any values from a specific level
Begin by seeing how many observations fall under each usertype
```{r}
table(all_trips$member_casual)
```
Reassign to the desired values (we will go with the current 2020 labels)

```{r}
all_trips <-  all_trips %>% 
  mutate(member_casual = recode(member_casual
                           ,"Subscriber" = "member"
                           ,"Customer" = "casual"))
```

Add columns that list the date, month, day, and year of each ride
This will allow us to aggregate ride data for each month, day, or year ... before completing these operations we could only aggregate at the ride level
See [statemethods](https://www.statmethods.net/input/dates.html) for more on date formats in R found at that link

```{r}
all_trips$date <- as.Date(all_trips$started_at) #The default format is yyyy-mm-dd
all_trips$month <- format(as.Date(all_trips$date), "%m")
all_trips$day <- format(as.Date(all_trips$date), "%d")
all_trips$year <- format(as.Date(all_trips$date), "%Y")
all_trips$day_of_week <- format(as.Date(all_trips$date), "%A")
```

Add a "ride_length" calculation to all_trips (in seconds)
See [stat.ethz](https://stat.ethz.ch/R-manual/R-devel/library/base/html/difftime.html) for more about this step

```{r}
all_trips$ride_length <- difftime(all_trips$ended_at,all_trips$started_at)
```

Inspect the structur of the columns

```{r}
str(all_trips)
```

Convert "ride_length" from Factor to numeric so we can run calculations on the data
```{r}
is.factor(all_trips$ride_length)
all_trips$ride_length <- as.numeric(as.character(all_trips$ride_length))
is.numeric(all_trips$ride_length)
```
Remove "bad" data
The dataframe includes a few hundred entries when bikes were taken out of docks and checked for quality by Divvy or ride_length was negative
We will create a new version of the dataframe (v2) since data is being removed
See mor under [datasciencemadesimple](https://www.datasciencemadesimple.com/delete-or-drop-rows-in-r-with-conditions-2/).
```{r}
all_trips_v2 <- all_trips[!(all_trips$start_station_name == "HQ QR" | all_trips$ride_length<0),]
```

### Conduct Descriptive Analysis

#### Descriptive analysis on ride_length (all figures in seconds)

```{r}
mean(all_trips_v2$ride_length) #straight average (total ride length / rides)
median(all_trips_v2$ride_length) #midpoint number in the ascending array of ride lengths
max(all_trips_v2$ride_length) #longest ride
min(all_trips_v2$ride_length) #shortest ride
```

You can condense the four lines above to one line using summary() on the specific attribute
```{r}
summary(all_trips_v2$ride_length)
```

#### Compare members and casual_riders

```{r}
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = mean)
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = median)
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = max)
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual, FUN = min)
```
#### See the average ride time by each day for members vs casual users

```{r}
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual + all_trips_v2$day_of_week, FUN = mean)
```

Notice that the days of the week are out of order. Let's fix that.

```{r}
all_trips_v2$day_of_week <- ordered(all_trips_v2$day_of_week, levels=c("Sonntag", "Montag", "Dienstag", "Mittwoch", "Donnerstag", "Freitag", "Samstag"))
```
Now, let's run the average ride time by each day for members vs casual users
```{r}
aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual + all_trips_v2$day_of_week, FUN = mean)
```

analyze ridership data by type and weekday
```{r}
all_trips_v2 %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>%  #creates weekday field using wday()
  group_by(member_casual, weekday) %>%  #groups by usertype and weekday
  summarise(number_of_rides = n()                            
          #calculates the number of rides and average duration 
  ,average_duration = mean(ride_length)) %>%         # calculates the average duration
  arrange(member_casual, weekday)
```

### Visualize the number of rides by rider type

```{r}
all_trips_v2 %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>% 
  group_by(member_casual, weekday) %>% 
  summarise(number_of_rides = n()
            ,average_duration = mean(ride_length)) %>% 
  arrange(member_casual, weekday)  %>% 
  ggplot(aes(x = weekday, y = number_of_rides, fill = member_casual)) +
  geom_col(position = "dodge")
```

### Visualization for average duration
```{r}
all_trips_v2 %>% 
  mutate(weekday = wday(started_at, label = TRUE)) %>% 
  group_by(member_casual, weekday) %>% 
  summarise(number_of_rides = n()
            ,average_duration = mean(ride_length)) %>% 
  arrange(member_casual, weekday)  %>% 
  ggplot(aes(x = weekday, y = average_duration, fill = member_casual)) +
  geom_col(position = "dodge")
```

### Export Summary File for further Analysis

Create a csv file that we will visualize in Excel, Tableau, or my presentation software
N.B.: This file location is for a Mac. If you are working on a PC, change the file location accordingly (most likely "C:\Users\YOUR_USERNAME\Desktop\...") to export the data. You can read more here: [datatofish](https://datatofish.com/export-dataframe-to-csv-in-r/)

```{r}
counts <- aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual + all_trips_v2$day, FUN = mean)
write.csv(counts, file = '~/2112-case_study_bicycle/avg_ride_length.csv')
```

### Summary

On weekends the number of rides for casual users and members tend to be even. Member usage is significantly going down while the trip duration time stays the same. This can be due to member users use the service more for there daily schedule.

### Further investigation

A closer look into the most used stations dependend on user ids might provide insights into the users preferences.
Investigate data about the type of rides members and casual users prefering can also reveal further insights.


### Recommendations

* Focus marketing efforts on casual riders for weekend activities
* Focus marketing efforts for long duration riders throughout the week
* Include data about types of rides into our analysis
