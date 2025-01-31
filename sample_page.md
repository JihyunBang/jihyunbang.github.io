## Case study: How does a bike-share navigate success? (R project)

**Synopsis: Cyclistic is a fictional bike-share company in Chicago, and the company's success depends on maximizing the number of annual memberships. To facilitate this, the team needs to understand the difference in trends between casual riders and annual members which require data insights and data visualizations. ** 

### 1. Install and load up required packages

To start, packages must be installed (if not already) and loaded up. I'll be using tidyverse, janitor, and lubridate. 

```javascript
install.packages('tidyverse')
install.packages('janitor')
install.packages('lubridate')

library(tidyverse)
library(janitor)
library(lubridate)
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

We will use the function ggplot() to create visualizations. We will put the weekdays on the x axis and the number of rides on the y axis. We will use the geom_col() function to make a bar graph that will represent members and casuals.

```javascript
all_trips_v2 %>%
	mutate(weekday = wday(started_at, label = TRUE)) %>%
	group_by(member_casual, weekday) %>%
	summarise(number_of_rides = n(), average_duration = mean(ride_length)) %>%
	arrange(member_casual, weekday) %>%
	ggplot(aes(x = weekday, y = number_of_rides, fill = member_casual)) + geom_col(position = "dodge")
```

This code will create a bar graph that looks like this:
<img src="https://github.com/JihyunBang/jihyunbang.github.io/blob/main/Screenshot%202025-01-07%20084739.png?raw=true">

### 17. Visualize # of rides by member and casual by month

We use the ggplot() function again, and we will assign the x axis as the month and the y axis as the number of rides. We will make a bar graph using the geom_col() function.

```javascript
all_trips_v2 %>%
	mutate(month = month(started_at, label = TRUE)) %>%
	group_by(member_casual, month) %>%
	summarise(number_of_rides = n(), average_duration = mean(ride_length)) %>%
	arrange(member_casual, month) %>%
	ggplot(aes(x = month, y = number_of_rides, fill = member_casual)) + geom_col(position = "dodge")
```

This code will create a bar graph that looks like this:
<img src="https://github.com/JihyunBang/jihyunbang.github.io/blob/main/Screenshot%202025-01-07%20084850.png?raw=true">

### 18. Visualize # of rides by member and casual by hours/day

Using the ggplot() function, we assign the x axis as the hours and the y axis as the number of rides. Using the geom_col() function, we make a bar graph. Using the facet_wrap() function, we make a multi-panel plot.

```javascript
all_trips_v2 %>%
	mutate(hours = hour(started_at)) %>%
	mutate(weekday = wday(started_at, label = TRUE)) %>%
	group_by(member_casual, hours, weekday) %>%
	summarise(number_of_rides = n(), average_duration = mean(ride_length)) %>%
	arrange(member_casual, hours) %>%
	ggplot(aes(x = hours, y = number_of_rides, fill = member_casual)) + geom_col(position = "dodge")+ facet_wrap(~weekday)
```

This code will create a bar graph that looks like this:
<img src="https://github.com/JihyunBang/jihyunbang.github.io/blob/main/Screenshot%202025-01-07%20084934.png?raw=true">

### 19. Visualize total # of rides by member and casual

With the ggplot() function, we assign the x axis as the type of user and the y axis as the number of rides. Using the geom_col() function, we make a bar graph.

```javascript
all_trips_v2 %>%
	group_by(member_casual)%>%
	summarise(number_of_rides = n(), average_duration = mean(ride_length)) %>%
	ggplot(aes(x = member_casual, y = number_of_rides, fill = member_casual)) + geom_col(position = "dodge")
```

This code will create a bar graph that looks like this:
<img src="https://github.com/JihyunBang/jihyunbang.github.io/blob/main/Screenshot%202025-01-07%20085002.png?raw=true">

### 20. Visualize average duration by member and casual

With the ggplot() function, we assign the x axis as the type of use and the y axis as the average duration. Using the geom_col() function, we make a bar graph.

```javascript
all_trips_v2 %>%
	group_by(member_casual)%>%
	summarise(number_of_rides = n(), average_duration = mean(ride_length)) %>%
	ggplot(aes(x = member_casual, y = average_duration, fill = member_casual)) + geom_col(position = "dodge")
```

This code will create a bar graph that looks like this:
<img src="https://github.com/JihyunBang/jihyunbang.github.io/blob/main/Screenshot%202025-01-07%20085442.png?raw=true">

### 21. Filter out casuals in data frame to find most popular stations by member

By using the filter() function, we are able to filter out casuals in the data to only return data related to members. First we group the data by using the group_by() function, then by using the summarize() function we can get the # of rides in each starting station. Then by arranging this data using the arrange() function in descending order, we can find the most frequently used/popular stations by members.

```javascript
member_all_trips_v2 <- filter(all_trips_v2, member_casual != "casual")

member_all_trips_v2 %>%
	group_by(member_casual, start_station_name)%>%
	summarize(count=n())%>%
	arrange(desc(count))
```

### 22. Filter out members in dataframe to find most popular stations by casual

By using the filter() function, we are able to filter out members in the data to only return data related to casuals. Like the previous step, we group the data by using the group_by() function, then by using the summarize() function we retrieve the # of rides in each starting station. Then we use the arrange() function in descending order to find the most popular stations by casuals.
```javascript
casual_all_trips_v2 <- filter(all_trips_v2, member_casual != "member")

casual_all_trips_v2 %>%
	group_by(member_casual, start_station_name)%>%
	summarize(count=n())%>%
	arrange(desc(count))
```

### 23. Find most popular stations in general

Since we are looking for the most popular stations in general, we do not need to use the filter() function, and we only group the data, summarize them, and arrange them in descending order.

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
**Conclusion: It seems that most members are workers as there is a peak of bike usage at around 8-9 A.M. and another peak at around 5 P.M. The total number of rides for casuals are lower but the average ride duration is higher which leads me to believe that there is a likely possibility that the casuals are consisted of tourists visiting the area.

Recommendations (Note: The director of marketing is focusing on maximizing the number of annual memberships)

1. To maximize the number of annual memberships, the company can implement a ride charge that is correlated to ride length/duration. The trend shows that the average ride duration is considerably higher in casuals. By increasing the ride charge by duration, the casual bike users would be inclined to purchase an annual membership.

2. Another recommendation to maximize the number of annual memberships is to host a seasonal sale. By doing a summer/spring/autumn sale, more casuals may be inclined to purchase an annual membership, and there may be a surge of profit increase during the seasonal sales.

3. Advertisements could also help in maximizing the number of annual memberships, and through the data we were able to find the most popular stations used by casual bike users (See step 22). By putting ads, we would be increasing the exposure of annual membership information to our target audience, which is the casual bike riders.**

To see a more in-depth, stylized visualization of this case study on TableauPublic: <https://public.tableau.com/app/profile/edward.bang/viz/CyclisticBike-CaseStudy/Dashboard1>
