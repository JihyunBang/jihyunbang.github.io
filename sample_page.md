## Case study: How does a bike-share navigate success? (R project)

**Synopsis: Cyclistic is a fictional bike-share company in Chicago, and the company's success depends on maximizing the number of annual memberships. To facilitate this, the team needs to understand the difference in trends between casual riders and annual members which require data insights and data visualizations. ** 

### 1. Install and load up required packages

To start, packages must be installed (if not already) and loaded up. I'll be using tidyverse, janitor, and lubridate. 

```javascript
install.packages('tidyverse')
install.packages('janitor')
install.packages('lubridate')
```

### 2. Upload data formatted in .csv files (ride id, ride type, start time, end time, start station, end station, coordinates, etc) and give them variables

```javascript
jan2023 <- read_csv("2023_01.csv")
feb2023 <- read_csv("2023_02.csv")
mar2023 <- read_csv("2023_02.csv")
...
```

### 3. To combine the data into a single file, the column names need to match, the function colnames() will help with this.

```javascript
colnames(jan2023)
colnames(feb2023)
colnames(mar2023)
...
```

### 4. To check for incongruencies, the function str() can be used. (Checks internal structure)

```javascript
str(jan2023)
str(feb2023)
str(mar2023)
...
```

 ### 5. The columns ride_id and rideable_type can be stacked if they are converted to character

 ```javascript
jan2023 <- mutate(jan2023, ride_id = as.character(ride_id),rideable_type = as.character(rideable_type))
```
### 6. Combine all data frames into a single variable

```
all_trips <- bind_rows(jan2023,feb2023,mar2023,apr2023,may2023,jun2023,jul2023,aug2023,spe2023,oct2023,nov2023,dec2023)
```

### 7. Remove lat and long columns (Columns that are not needed can be removed this way)

```javascript
all_trips <- all_trips %>%
  select(-c(start_lat,start_lng,end_lat,end_lng))
```

### 8. Check colnames, # of rows, data dimensions, first rows, last rows, list of cols and data types, and summary to check information

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

```javascript
all_trips <- all_trips %>%
  mutate(member_casual = recode(member_casual,"Subscriber" = "member", "Customer" = "casual"))
```

### 10. Create columns for date, month, day and year

```javascript
all_trips$date <- as.Date(all_trips$started_at)
all_trips$month <- format(as.Date(all_trips$date), "%m")
all_trips$day <- format(as.Date(all_trips$date), "%d")
all_trips$year <- format(as.Date(all_trips$date), "%Y")
all_trips$day_of_week <- format(as.Date(all_trips$date), "%A")
all_trips$hour <- format(as.POSIXct(all_trips$date), "%H")
```

### 11. Create ride_length calculation by finding difftime and check structure again

```javascript
all_trips$ride_length <- difftime(all_trips$ended,at,all_trips$started_at)

str(all_trips)
```

### 12. Check ride_length, if it's a factor, convert ride_length from factor to numeric

```javascript
is.factor(all_trips$ride_length)
all_trips$ride_length <- as.numeric(as.character(all_trips$ride_length))
is.numeric(all_trips$ride_length)
```

### 13. Creating a dataframe that doesn't have bad data, and check summary of the new dataframe

```javascript
all_trips_v2 <- all_trips[!(all_trips$start_station_name == "HQ QR" | all_trips$ride_length<0),]
all_trips_v2 <- all_trips_v2[complete.cases(all_trips_v2),]

summary(all_trips_v2$ride_length)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).
