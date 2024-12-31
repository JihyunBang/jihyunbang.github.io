## Case study: How does a bike-share navigate success? (R project)

**Synopsis: Cyclistic is a fictional bike-share company in Chicago, and the company's success depends on maximizing the number of annual memberships. To facilitate this, the team needs to understand the difference in trends between casual riders and annual members which require data insights and data visualizations. ** 

### 1. Install and load up required packages

To start, packages must be installed (if not already) and loaded up. I'll be using tidyverse, janitor, and lubridate. 

```javascript
install.packages('tidyverse')
install.packages('janitor')
install.packages('lubridate')
```

### 2. Upload data formatted in .csv files

To upload data, we use read_csv() and assign it to a variable, such as "jan2023", "feb2023", and so on. The dataframe contains columns such as ride id, ride type, start time, start station, end station, coordinates, etc.

```javascript
jan2023 <- read_csv("2023_01.csv")
feb2023 <- read_csv("2023_02.csv")
mar2023 <- read_csv("2023_02.csv")
...
```

### 3. Combine the data into a single file

To combine data, the column names need to match. We can check for incongruencies with colnames().

```javascript
colnames(jan2023)
colnames(feb2023)
colnames(mar2023)
...
```

### 4. Check for incongruencies

To check for incongruencies in the internal structure, we can use the function str().

```javascript
str(jan2023)
str(feb2023)
str(mar2023)
...
```

### 5. Convert columns to character

The columns ride_id and rideable_type can be stacked if they are converted to character using the mutate() and as.character() functions.

```javascript
jan2023 <- mutate(jan2023, ride_id = as.character(ride_id),rideable_type = as.character(rideable_type))
```
### 6. Combine all data frames into a single variable

To combine data frames, we can use the function bind_rows()

```
all_trips <- bind_rows(jan2023,feb2023,mar2023,apr2023,may2023,jun2023,jul2023,aug2023,spe2023,oct2023,nov2023,dec2023)
```

### 7. Remove unnecessary columns

We can drop columns by name using the function select(-c()). In this case study, the latitude and longitude values are not needed.

```javascript
all_trips <- all_trips %>%
  select(-c(start_lat,start_lng,end_lat,end_lng))
```

### 8. Check various data information

We can do this with the functions colnames() to check column names, nrow() to check # of rows, dim() to check dimensions, head() to check first rows, tail() to check last rows, str() to check list of columns and data types, and summary() to check the summary.

```javascript
colnames(all_trips)
nrow(all_trips)
dim(all_trips)
head(all_trips)
tail(all_trips)
str(all_trips)
summary(all_trips)
```

### 9. Change "Subscriber" to "Member", and "Customer" to "Casual"

We can achieve this by using the mutate() function to modify the existing columns.

```javascript
all_trips <- all_trips %>%
  mutate(member_casual = recode(member_casual,"Subscriber" = "member", "Customer" = "casual"))
```

### 10. Create columns for date, month, day and year

This can be achieved through the as.Date() function to convert character objects to date objects. To store a date and time to find the hour, the function as.POSIXct() can be used. In the code below, %m, %d, %Y %A and %H represents month, day, year, day of week, and hour, respectively.

```javascript
all_trips$date <- as.Date(all_trips$started_at)
all_trips$month <- format(as.Date(all_trips$date), "%m")
all_trips$day <- format(as.Date(all_trips$date), "%d")
all_trips$year <- format(as.Date(all_trips$date), "%Y")
all_trips$day_of_week <- format(as.Date(all_trips$date), "%A")
all_trips$hour <- format(as.POSIXct(all_trips$date), "%H")
```

### 11. Calculate ride length

We can do this by using the function difftime() which is a function that can be used to return the difference of two times.

```javascript
all_trips$ride_length <- difftime(all_trips$ended,at,all_trips$started_at)

str(all_trips)
```

### 12. Convert ride_length from factor to numeric

To check if ride_length is factor is the first place, we can use the function is.factor() which will return a value of TRUE or FALSE. To convert this to a numeric object, we can implement the function as.numeric(), and use the function is.numeric() function to verify if it was correctly converted.

```javascript
is.factor(all_trips$ride_length)
all_trips$ride_length <- as.numeric(as.character(all_trips$ride_length))
is.numeric(all_trips$ride_length)
```

### 13. Remove bad data

We can do this by creating a new dataframe that doesn't have bad data by removing invalid station names, and ride length that is less than zero.

```javascript
all_trips_v2 <- all_trips[!(all_trips$start_station_name == "HQ QR" | all_trips$ride_length<0),]
all_trips_v2 <- all_trips_v2[complete.cases(all_trips_v2),]

summary(all_trips_v2$ride_length)
```

### 14. Calculate average ride time and rearrange days of the week

The aggregate() function will help with computing summary statistics to find the average ride time and the ordered() function will order the days of the week.

```javascript
aggregate(all_trips_v2$ride_length~all_trips_v2$member_casual + all_trips_v2$day_of_week, FUN=mean)

all_trips_v2$day_of_week <- ordered(all_trips_v2$day_of_week, levels=c("Sunday","Monday","Tuesday","Wednesday","Thursday","Friday","Saturday"))
```

### 15. Arrange rider data

To achieve this, we can use the mutate() function to create a column (weekday, month and hours in this code), then use the group_by() function to group the data. We can then summarise this data and retrieve the number of rides and mean ride length, then order the output by using the arrange() function.

```javascript
all_trips_v2 %>%
	mutate(weekday = wday(started_at, label = TRUE)) %>%
	group_by(member_casual, weekday) %>%
	summarise(number_of_rides = n(), average_duration = mean(ride_length))%>%
	arrange(member_casual, weekday)
all_trips_v2 %>%
	mutate(month = month(started_at, label = TRUE)) %>%
	group_by(member_casual, month) %>%
	summarise(number_of_rides = n(), average_duration = mean(ride_length))%>%
	arrange(member_casual, month)

all_trips_v2 %>%
	mutate(hours = hour(started_at)) %>%
	group_by(member_casual,hours,day_of_week)%>%
	summarise(number_of_rides = n(), average_duration = mean(ride_length))%>%
	arrange(member_casual, hours)
```

### 16. Visualize # of rides by member and casual by weekday

```javascript
all_trips_v2 %>%
	mutate(weekday = wday(started_at, label = TRUE)) %>%
	group_by(member_casual, weekday) %>%
	summarise(number_of_rides = n(), average_duration = mean(ride_length)) %>%
	arrange(member_casual, weekday) %>%
	ggplot(aes(x = weekday, y = number_of_rides, fill = member_casual)) + geom_col(position = "dodge")
```

### 17. Visualize # of rides by member and casual by month

```javascript
all_trips_v2 %>%
	mutate(month = month(started_at, label = TRUE)) %>%
	group_by(member_casual, month) %>%
	summarise(number_of_rides = n(), average_duration = mean(ride_length)) %>%
	arrange(member_casual, month) %>%
	ggplot(aes(x = month, y = number_of_rides, fill = member_casual)) + geom_col(position = "dodge")
```

### 18. Visualize # of rides by member and casual by hours/day

```javascript
all_trips_v2 %>%
	mutate(hours = hour(started_at)) %>%
	mutate(weekday = wday(started_at, label = TRUE)) %>%
	group_by(member_casual, hours, weekday) %>%
	summarise(number_of_rides = n(), average_duration = mean(ride_length)) %>%
	arrange(member_casual, hours) %>%
	ggplot(aes(x = hours, y = number_of_rides, fill = member_casual)) + geom_col(position = "dodge")+ facet_wrap(~weekday)
```

### 19. Visualize total # of rides by member and casual

```javascript
all_trips_v2 %>%
	group_by(member_casual)%>%
	summarise(number_of_rides = n(), average_duration = mean(ride_length)) %>%
	ggplot(aes(x = member_casual, y = number_of_rides, fill = member_casual)) + geom_col(position = "dodge")
```

### 20. Visualize average duration by member and casual

```javascript
all_trips_v2 %>%
	group_by(member_casual)%>%
	summarise(number_of_rides = n(), average_duration = mean(ride_length)) %>%
	ggplot(aes(x = member_casual, y = average_duration, fill = member_casual)) + geom_col(position = "dodge")
```

### 21. Filter out casuals in data frame to find most popular stations by member

```javascript
member_all_trips_v2 <- filter(all_trips_v2, member_casual != "casual")

member_all_trips_v2 %>%
	group_by(member_casual, start_station_name)%>%
	summarize(count=n())%>%
	arrange(desc(count))
```

### 22. Filter out members in dataframe to find most popular stations by casual
```javascript
casual_all_trips_v2 <- filter(all_trips_v2, member_casual != "member")

casual_all_trips_v2 %>%
	group_by(member_casual, start_station_name)%>%
	summarize(count=n())%>%
	arrange(desc(count))
```

### 23. Find most popular stations in general
```javascript
all_trips_v2 %>%
	group_by(member_casual, start_station_name)%>%
	summarize(count=n())%>%
	arrange(desc(count))
```

### 24. Export as csv file
```javascript
counts <- aggregate(all_trips_v2$ride_length ~ all_trips_v2$member_casual + all_trips_v2$day_of_week, FUN = mean)
write.csv(counts, file = 'avg_ride_length.csv')
```
To see a more in-depth, stylized visualization of this case study on TableauPublic: <https://public.tableau.com/app/profile/edward.bang/viz/CyclisticBike-CaseStudy/Dashboard1>
