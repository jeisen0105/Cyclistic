# Case Study: Cyclistic Bike-Share Analysis (Google Data Analytics Capstone)

## Introduction

This project analyzes the Cyclistic Bike-Share case study, developed as part of the Google Data Analytics Professional Certificate capstone. The objective is to answer key business questions by applying the six-step data analysis process: Ask, Prepare, Process, Analyze, Share, and Act.

## Background

Cyclistic is a bike-share program based in Chicago, offering 5,824 bicycles and 692 docking stations throughout the city. Unlike many competitors, Cyclistic provides a variety of bicycle types—including reclining bikes, hand tricycles, and cargo bikes—designed to accommodate riders with different needs, making the service more inclusive and accessible.

The company offers flexible pricing plans, including single-ride passes, full-day passes, and annual memberships. While this approach appeals to a broad range of users, the company’s marketing director believes that long-term success depends on increasing the number of annual members.

## Scenario

For the purposes of this case study, I assume the role of a junior data analyst on Cyclistic’s marketing analytics team. The objective is to analyze rider behavior and present findings and recommendations to key stakeholders, including Marketing Director Lily Moreno and the Cyclistic executive team.
 
## Step 1: Ask

The business task is to analyze how annual members and casual riders use Cyclistic bikes differently. By identifying usage patterns over time, the marketing team can develop targeted strategies aimed at converting more casual riders into annual members. 

## Step 2: Prepare

### Does the Data ROCCC?

To ensure the quality of this analysis, the data must be Reliable, Original, Comprehensive, Current, and Cited (ROCCC). The dataset used in this project consists of historical trip data from Divvy, Chicago’s bike-share system, from September 8th 2024 to September 8th 2025. The data was made publicly available by Motivate International Inc. under their Data License Agreement. The data is also public and anonymized to protect riders' privacy. It excludes any personally identifiable information such as names, phone numbers, or payment details.

Applying the ROCCC framework:
- Reliable & Original: The data comes directly from a primary source (Divvy) and reflects actual trip history.
- Comprehensive: It includes all necessary fields to analyze usage patterns between member types.
- Current: Data is from September 2024-Septemeber 2025 which is very current.
- Cited: The data source is publicly documented and licensed appropriately.

### Preparing RStudio

To start the analysis, we first set up the R environment in RStudio. We installed and loaded the tidyverse package, which gives us a powerful collection of tools for manipulating and visualizing the data.

```r
install.packages("tidyverse")

library(tidyverse)
```

### Data Consolidation

All raw data, which were stored in individual CSV files, were imported and combined into a single datafrane. The analysis identified all files with a .csv extension in the project directory, then read each one and combined them row-wise into a final data frame called combined_data.

```r
all_files <- list.files(
  path = ".", 
  pattern = "\\.csv$", 
  full.names = TRUE 
)

combined_data <- map_dfr(all_files, read_csv)
```

## Step 3: Process

### Data Cleaning 

Unnecessary columns were removed, and duplicate rows were eliminated to ensure data integrity. Missing values were also removed to prevent them from skewing the analysis.

```r
combined_data <- combined_data %>%
  select(-rideable_type, 
         -start_station_id, 
         -end_station_id, 
         -start_lat, 
         -start_lng, 
         -end_lat, 
         -end_lng)

combined_data <- combined_data %>%
  distinct() %>% 
  drop_na()
```

### Data Transformation

New columns were created from the existing timestamp data to extract useful information for the analysis. This included extracting the date, month, day_of_week, year, hour_of_day, and calculating ride_length in minutes.

```r
combined_data <- combined_data %>%
  mutate(
    date = as.Date(started_at), 
    month = lubridate::month(started_at, label = TRUE), 
    day = lubridate::day(started_at), 
    year = lubridate::year(started_at), 
    day_of_week = lubridate::wday(started_at, label = TRUE),
    hour_of_day = lubridate::hour(started_at), 
    ride_length = round(as.numeric(difftime(ended_at, started_at, units = "mins")), 2)
  )
```

### Data Filtering

The final step in the process phase involved filtering the dataset to remove any invalid ride entries. This included rides with a duration of zero or less, as well as those that were excessively long (over 24 hours), ensuring that the analysis focused on valid user activity.

```r
combined_data <- combined_data %>%
  filter(ride_length > 0 & ride_length < 1440)
```

## Step 4: Analyze

### Descriptive Analysis on Ride Length by User Type

After cleaning and manipulating the data, descriptive statistics were calculated to understand ride duration based on membership type. The mean, median, minimum, and maximum ride lengths were summarized for both "member" and "casual" riders. The results of this summary were then saved to a CSV file named ride_duration_stats.csv for further reference

```r
ride_duration_stats <- combined_data %>%
  group_by(member_casual) %>%
  summarize(
    mean_ride_length = round(mean(ride_length), 2),
    median_ride_length = round(median(ride_length), 2),
    min_ride_length = round(min(ride_length), 2),
    max_ride_length = round(max(ride_length), 2)
  )

write_csv(ride_duration_stats, "ride_duration_stats.csv")
```

The table below shows the descriptive statistics for ride length in minutes, comparing casual and annual members. Specifically, it summarizes the mean, median, minimum, and maximum ride duration for each user type.

| member_casual 	| mean_ride_length 	| median_ride_length 	| min_ride_length 	| max_ride_length 	|
|---------------	|------------------	|--------------------	|-----------------	|-----------------	|
| casual        	| 22.78            	| 13.02              	| 0.01            	| 1439.76         	|
| member        	| 12.19            	| 8.72               	| 0.01            	| 1436.97         	|

### Analyzing Daily Usage Trend by User Type

Further analysis was performed to understand daily rider behavior. The data was grouped by both member type and day of the week to calculate the total number of rides and the average ride length. This summary was then saved to a CSV file named ride_duration_and_count_by_weekday.csv for documentation and further analysis.

```r
ride_duration_and_count_by_weekday <- combined_data %>%
  group_by(member_casual, day_of_week) %>%
  summarize(
    total_rides = n(),
    average_ride_length = round(mean(ride_length), 2)
  )

write_csv(ride_duration_and_count_by_weekday, "ride_duration_and_count_by_weekday.csv")
```

The table below shows the daily usage patterns for casual and member riders. It presents the total number of rides and the average ride length (in minutes), aggregated by member type and day of the week.

| member_casual 	| day_of_week 	| total_rides 	| average_ride_length 	|
|---------------	|-------------	|-------------	|---------------------	|
| casual        	| Sun         	| 284143      	| 26.11               	|
| casual        	| Mon         	| 188447      	| 21.99               	|
| casual        	| Tue         	| 175543      	| 19.93               	|
| casual        	| Wed         	| 180085      	| 19.27               	|
| casual        	| Thu         	| 202471      	| 20.01               	|
| casual        	| Fri         	| 254276      	| 22.25               	|
| casual        	| Sat         	| 340219      	| 25.78               	|
| member        	| Sun         	| 302963      	| 13.63               	|
| member        	| Mon         	| 405925      	| 11.65               	|
| member        	| Tue         	| 434190      	| 11.7                	|
| member        	| Wed         	| 430217      	| 11.69               	|
| member        	| Thu         	| 437814      	| 11.72               	|
| member        	| Fri         	| 402533      	| 12.08               	|
| member        	| Sat         	| 341605      	| 13.52               	|

### Analyzing Monthly Usage Trends by User Type

To further explore rider behavior, a summary was created to analyze monthly trends in ride duration and count. The data was grouped by member type and month to calculate the total number of rides and the average ride length. This summary was then saved to a CSV file named ride_duration_and_count_by_month.csv

```r
ride_duration_and_count_by_month <- combined_data %>%
  group_by(member_casual, month) %>%
  summarize(
    total_rides = n(),
    average_ride_length = round(mean(ride_length), 2)
  )

write_csv(ride_duration_and_count_by_month, "ride_duration_and_count_by_month.csv")
```
The table below shows the monthly trends in ridership, summarizing the total number of rides and the average ride length (in minutes), categorized by member type and month.

| member_casual 	| month 	| total_rides 	| average_ride_length 	|
|---------------	|-------	|-------------	|---------------------	|
| casual        	| Jan   	| 17090       	| 12.94               	|
| casual        	| Feb   	| 19631       	| 13.44               	|
| casual        	| Mar   	| 61667       	| 19.65               	|
| casual        	| Apr   	| 77014       	| 20.79               	|
| casual        	| May   	| 125685      	| 23.32               	|
| casual        	| Jun   	| 193111      	| 24.53               	|
| casual        	| Jul   	| 208821      	| 24.29               	|
| casual        	| Aug   	| 449650      	| 24.48               	|
| casual        	| Sep   	| 216132      	| 22.08               	|
| casual        	| Oct   	| 159340      	| 22.32               	|
| casual        	| Nov   	| 68795       	| 17.75               	|
| casual        	| Dec   	| 28248       	| 14.91               	|
| member        	| Jan   	| 84121       	| 9.91                	|
| member        	| Feb   	| 89945       	| 9.86                	|
| member        	| Mar   	| 148160      	| 11.15               	|
| member        	| Apr   	| 181857      	| 11.37               	|
| member        	| May   	| 215008      	| 12.11               	|
| member        	| Jun   	| 253190      	| 13.08               	|
| member        	| Jul   	| 285590      	| 13.47               	|
| member        	| Aug   	| 607118      	| 13.15               	|
| member        	| Sep   	| 320853      	| 12.27               	|
| member        	| Oct   	| 289761      	| 11.88               	|
| member        	| Nov   	| 177129      	| 10.95               	|
| member        	| Dec   	| 102515      	| 10.51               	|

### Analyzing Hourly Usage Trends by User Type

To provide a more granular view of daily ridership, the data was further summarized to analyze hourly trends. The data was grouped by member type and hour of the day, allowing for the calculation of total rides and average ride length for each hour. This summary was then saved to a CSV file named ride_duration_and_count_by_hour.csv.

```r
ride_duration_and_count_by_hour <- combined_data %>%
  group_by(member_casual, hour_of_day) %>%
  summarize(
    total_rides = n(),
    average_ride_length = round(mean(ride_length), 2)
  )

write_csv(ride_duration_and_count_by_hour, "ride_duration_and_count_by_hour.csv")
```
The table below shows a granular view of daily ridership, summarizing the total number of rides and the average ride length (in minutes), broken down by member type and hour of the day (0−23).

| member_casual 	| hour_of_day 	| total_rides 	| average_ride_length 	|
|---------------	|-------------	|-------------	|---------------------	|
| casual        	| 0           	| 26870       	| 19.72               	|
| casual        	| 1           	| 17282       	| 20.32               	|
| casual        	| 2           	| 10472       	| 19.88               	|
| casual        	| 3           	| 5733        	| 19.56               	|
| casual        	| 4           	| 4635        	| 17.55               	|
| casual        	| 5           	| 8923        	| 15.47               	|
| casual        	| 6           	| 22289       	| 14.96               	|
| casual        	| 7           	| 41800       	| 15.44               	|
| casual        	| 8           	| 59269       	| 17.01               	|
| casual        	| 9           	| 59730       	| 23.68               	|
| casual        	| 10          	| 74182       	| 27.79               	|
| casual        	| 11          	| 94582       	| 27.84               	|
| casual        	| 12          	| 109374      	| 26.39               	|
| casual        	| 13          	| 112375      	| 26.52               	|
| casual        	| 14          	| 116963      	| 25.86               	|
| casual        	| 15          	| 129771      	| 24.1                	|
| casual        	| 16          	| 146638      	| 22.2                	|
| casual        	| 17          	| 155537      	| 21.5                	|
| casual        	| 18          	| 128649      	| 21.26               	|
| casual        	| 19          	| 93596       	| 21.37               	|
| casual        	| 20          	| 66827       	| 20.34               	|
| casual        	| 21          	| 56041       	| 19.91               	|
| casual        	| 22          	| 48786       	| 19.79               	|
| casual        	| 23          	| 34860       	| 20.17               	|
| member        	| 0           	| 20660       	| 11.99               	|
| member        	| 1           	| 12488       	| 12.2                	|
| member        	| 2           	| 6916        	| 12.56               	|
| member        	| 3           	| 4700        	| 12.32               	|
| member        	| 4           	| 6072        	| 11.48               	|
| member        	| 5           	| 27044       	| 11.05               	|
| member        	| 6           	| 84006       	| 10.76               	|
| member        	| 7           	| 166843      	| 11.07               	|
| member        	| 8           	| 205421      	| 10.98               	|
| member        	| 9           	| 132497      	| 11.09               	|
| member        	| 10          	| 113533      	| 12.04               	|
| member        	| 11          	| 132028      	| 12.2                	|
| member        	| 12          	| 148596      	| 11.88               	|
| member        	| 13          	| 145592      	| 12                  	|
| member        	| 14          	| 147382      	| 12.48               	|
| member        	| 15          	| 184677      	| 12.53               	|
| member        	| 16          	| 262995      	| 12.7                	|
| member        	| 17          	| 300003      	| 13.02               	|
| member        	| 18          	| 224161      	| 12.83               	|
| member        	| 19          	| 154079      	| 12.78               	|
| member        	| 20          	| 104971      	| 12.57               	|
| member        	| 21          	| 79932       	| 12.52               	|
| member        	| 22          	| 56182       	| 12.48               	|
| member        	| 23          	| 34469       	| 12.62               	|

### Analyzing Top 10 Start Stations by User Type

To understand ridership patterns across different station locations, the top 10 most popular starting stations were identified and compared between casual and member riders. The data was first counted by member type and station name, then grouped by member type, and finally, the top 10 stations with the highest total ride count were selected for each group. The final result was sorted and saved to a CSV file named top_10_starting_stations.csv

```r
top_10_starting_stations <- combined_data %>%
  count(member_casual, start_station_name, name = "total_rides") %>%
  group_by(member_casual) %>%
  slice_max(order_by = total_rides, n = 10) %>%
  arrange(member_casual, desc(total_rides))

write_csv(top_10_starting_stations, "top_10_starting_stations.csv")
```
The table below shows the top 10 most popular starting stations for each user segment, displaying the station name and the total number of rides originating from that station for both casual and annual members.

| member_casual 	| start_station_name                 	| total_rides 	|
|---------------	|------------------------------------	|-------------	|
| casual        	| Streeter Dr & Grand Ave            	| 42986       	|
| casual        	| DuSable Lake Shore Dr & Monroe St  	| 35010       	|
| casual        	| Michigan Ave & Oak St              	| 24944       	|
| casual        	| Millennium Park                    	| 22191       	|
| casual        	| DuSable Lake Shore Dr & North Blvd 	| 22148       	|
| casual        	| Shedd Aquarium                     	| 20470       	|
| casual        	| Dusable Harbor                     	| 17733       	|
| casual        	| Theater on the Lake                	| 17160       	|
| casual        	| Michigan Ave & 8th St              	| 13189       	|
| casual        	| Navy Pier                          	| 12023       	|
| member        	| Kingsbury St & Kinzie St           	| 31061       	|
| member        	| Clinton St & Washington Blvd       	| 25566       	|
| member        	| Clinton St & Madison St            	| 23142       	|
| member        	| Clark St & Elm St                  	| 23117       	|
| member        	| Canal St & Madison St              	| 21140       	|
| member        	| Clinton St & Jackson Blvd          	| 19474       	|
| member        	| State St & Chicago Ave             	| 18580       	|
| member        	| Wells St & Elm St                  	| 18491       	|
| member        	| Wells St & Concord Ln              	| 18105       	|
| member        	| University Ave & 57th St           	| 16866       	|







