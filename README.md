
<!-- README.md is generated from README.Rmd. Please edit the README.Rmd file -->

# Lab report \#4 - instructions

Follow the instructions posted at
<https://ds202-at-isu.github.io/labs.html> for the lab assignment. The
work is meant to be finished during the lab time, but you have time
until Monday (after Thanksgiving) to polish things.

All submissions to the github repo will be automatically uploaded for
grading once the due date is passed. Submit a link to your repository on
Canvas (only one submission per team) to signal to the instructors that
you are done with your submission.

# Lab 4: Scraping (into) the Hall of Fame

![](README_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

``` r
library(rvest)
```

    ## Warning: package 'rvest' was built under R version 4.4.3

    ## 
    ## Attaching package: 'rvest'

    ## The following object is masked from 'package:readr':
    ## 
    ##     guess_encoding

``` r
library(Lahman)
```

    ## Warning: package 'Lahman' was built under R version 4.4.3

\##Step 1: Scrape data

I got the data by scraping the given URL, reading it as an HTML, and
getting the table we wanted with HOF induction information.

``` r
url <- "https://www.baseball-reference.com/awards/hof_2025.shtml"
html <- read_html(url)
tables <- html %>% html_table(fill = TRUE)
data <- tables[[1]]
```

\##Step 2: Clean data

First, I got the column names from the first record in the table and
made them the column names for the data frame. Then, selected the
columns that I would need to add to the Lahman data (yearID, votedBy,
ballots, needed, votes, inducted, category, needed_note). I then changed
the data type of `Votes` into numeric. Some names had an “X-” in front
of them, so I removed them to get just the names.

``` r
actual_col_names <- data[1, ]
colnames(data) <- actual_col_names
data <- data[-1, ]

head(HallOfFame)
```

    ##    playerID yearID votedBy ballots needed votes inducted category needed_note
    ## 1 aaronha01   1982   BBWAA     415    312   406        Y   Player        <NA>
    ## 2 abbotji01   2005   BBWAA     516    387    13        N   Player        <NA>
    ## 3 abreubo01   2020   BBWAA     397    298    22        N   Player        <NA>
    ## 4 abreubo01   2021   BBWAA     401    301    35        N   Player        <NA>
    ## 5 abreubo01   2022   BBWAA     394    296    34        N   Player        <NA>
    ## 6 abreubo01   2023   BBWAA     389    292    60        N   Player        <NA>

``` r
data <- data %>% select(c("Name", "Votes"))
data$Votes <- as.numeric(data$Votes)
data$Name <- gsub("X-", "", data$Name)
data
```

    ## # A tibble: 28 × 2
    ##    Name            Votes
    ##    <chr>           <dbl>
    ##  1 Ichiro Suzuki     393
    ##  2 CC Sabathia       342
    ##  3 Billy Wagner      325
    ##  4 Carlos Beltrán    277
    ##  5 Andruw Jones      261
    ##  6 Chase Utley       157
    ##  7 Alex Rodriguez    146
    ##  8 Manny Ramirez     135
    ##  9 Andy Pettitte     110
    ## 10 Félix Hernández    81
    ## # ℹ 18 more rows

Next, I got the player ID and names of people in the People data in
Lahman’s package. This needs to be joined with the new data. This caused
a duplicate

``` r
peopleneeded <- People %>% mutate(
  Name = paste(`nameFirst`, `nameLast`)
) %>% select(playerID, Name)
```

After checking with the anti-join, I noticed that some people may be
named differently in the data than in Lahman’s package, namely Carlos
Beltran, Felix Hernandez, Francisco Rodriguez, Carlos Gonzalez, and
Hanley Ramirez.

``` r
data %>% anti_join(peopleneeded, by = "Name")
```

    ## # A tibble: 5 × 2
    ##   Name                Votes
    ##   <chr>               <dbl>
    ## 1 Carlos Beltrán        277
    ## 2 Félix Hernández        81
    ## 3 Francisco Rodríguez    40
    ## 4 Carlos González         2
    ## 5 Hanley Ramírez          0

``` r
People %>% filter(nameFirst %in% c("Carlos", "Felix", "Francisco", "Hanley") & nameLast %in% c("Beltran", "Hernandez", "Rodriguez", "Gonzalez", "Ramirez")) %>% select(nameFirst, nameLast)
```

    ##    nameFirst  nameLast
    ## 1     Carlos   Beltran
    ## 2     Carlos  Gonzalez
    ## 3     Carlos Hernandez
    ## 4     Carlos Hernandez
    ## 5     Carlos Hernandez
    ## 6     Carlos Hernandez
    ## 7      Felix Hernandez
    ## 8     Carlos   Ramirez
    ## 9     Hanley   Ramirez
    ## 10    Carlos Rodriguez
    ## 11     Felix Rodriguez
    ## 12 Francisco Rodriguez
    ## 13 Francisco Rodriguez

I realized that’s probably because of the accents in their names in the
new data, so I replaced those with the regular letters. Then, anti-join
resulted in 0 missing people, so I used a left-join on the new data and
the people needed data. This caused a duplicate, since there were two
Francisco Rodriguez’s in the People dataset. After doing some research
on the Francisco Rodriguez in the website we used, the correct one had
the player ID “rodrifr03”, so I kept that one in our data.

``` r
data$Name <- gsub("á", "a", data$Name)
data$Name <- gsub("é", "e", data$Name)
data$Name <- gsub("í", "i", data$Name)

data %>% anti_join(peopleneeded, by = "Name")
```

    ## # A tibble: 0 × 2
    ## # ℹ 2 variables: Name <chr>, Votes <dbl>

``` r
data <- data %>% left_join(
  peopleneeded %>% select(Name, playerID), 
  by = "Name")

data <- data %>% filter(playerID != "rodrifr04")
data
```

    ## # A tibble: 28 × 3
    ##    Name            Votes playerID 
    ##    <chr>           <dbl> <chr>    
    ##  1 Ichiro Suzuki     393 suzukic01
    ##  2 CC Sabathia       342 sabatcc01
    ##  3 Billy Wagner      325 wagnebi02
    ##  4 Carlos Beltran    277 beltrca01
    ##  5 Andruw Jones      261 jonesan01
    ##  6 Chase Utley       157 utleych01
    ##  7 Alex Rodriguez    146 rodrial01
    ##  8 Manny Ramirez     135 ramirma02
    ##  9 Andy Pettitte     110 pettian01
    ## 10 Felix Hernandez    81 hernafe02
    ## # ℹ 18 more rows

Next, I changed my data to be the same format at the HallOfFame data.

``` r
head(HallOfFame)
```

    ##    playerID yearID votedBy ballots needed votes inducted category needed_note
    ## 1 aaronha01   1982   BBWAA     415    312   406        Y   Player        <NA>
    ## 2 abbotji01   2005   BBWAA     516    387    13        N   Player        <NA>
    ## 3 abreubo01   2020   BBWAA     397    298    22        N   Player        <NA>
    ## 4 abreubo01   2021   BBWAA     401    301    35        N   Player        <NA>
    ## 5 abreubo01   2022   BBWAA     394    296    34        N   Player        <NA>
    ## 6 abreubo01   2023   BBWAA     389    292    60        N   Player        <NA>

``` r
data <- data %>% mutate(
  yearID = 2025,
  votedBy = "BBWAA",
  ballots = 394,
  needed = 296,
  inducted = ifelse(Votes>=296, "Y", "N"),
  category = "Player",
  needed_note = NA
) %>% select(-Name) %>% 
  rename(
  votes = Votes
)

data
```

    ## # A tibble: 28 × 9
    ##    votes playerID  yearID votedBy ballots needed inducted category needed_note
    ##    <dbl> <chr>      <dbl> <chr>     <dbl>  <dbl> <chr>    <chr>    <lgl>      
    ##  1   393 suzukic01   2025 BBWAA       394    296 Y        Player   NA         
    ##  2   342 sabatcc01   2025 BBWAA       394    296 Y        Player   NA         
    ##  3   325 wagnebi02   2025 BBWAA       394    296 Y        Player   NA         
    ##  4   277 beltrca01   2025 BBWAA       394    296 N        Player   NA         
    ##  5   261 jonesan01   2025 BBWAA       394    296 N        Player   NA         
    ##  6   157 utleych01   2025 BBWAA       394    296 N        Player   NA         
    ##  7   146 rodrial01   2025 BBWAA       394    296 N        Player   NA         
    ##  8   135 ramirma02   2025 BBWAA       394    296 N        Player   NA         
    ##  9   110 pettian01   2025 BBWAA       394    296 N        Player   NA         
    ## 10    81 hernafe02   2025 BBWAA       394    296 N        Player   NA         
    ## # ℹ 18 more rows

Finally, I binded the datasets and got the CSV file.

``` r
newdf <- rbind(data, HallOfFame)
readr::write_csv(newdf, file="HallOfFame.csv")
```
