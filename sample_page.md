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

Sed ut perspiciatis unde omnis iste natus error sit voluptatem accusantium doloremque laudantium, totam rem aperiam, eaque ipsa quae ab illo inventore veritatis et quasi architecto beatae vitae dicta sunt explicabo. 

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).
