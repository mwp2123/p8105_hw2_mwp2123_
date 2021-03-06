Homework 2
================

``` r
library(tidyverse)
```

    ## ── Attaching packages ──────────────────────────────────────────────────────────────── tidyverse 1.3.0 ──

    ## ✓ ggplot2 3.3.2     ✓ purrr   0.3.4
    ## ✓ tibble  3.0.3     ✓ dplyr   1.0.2
    ## ✓ tidyr   1.1.2     ✓ stringr 1.4.0
    ## ✓ readr   1.3.1     ✓ forcats 0.5.0

    ## ── Conflicts ─────────────────────────────────────────────────────────────────── tidyverse_conflicts() ──
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(readxl)
```

## Problem 1

Read the Mr. Trashwheel dataset

``` r
trashwheel_df = 
  read_excel(
    "./data/Trash-Wheel-Collection-Totals-8-6-19.xlsx", 
    sheet = "Mr. Trash Wheel", 
    range = cell_cols("A:N")) %>% 
  janitor::clean_names() %>% 
  drop_na(dumpster) %>% 
  mutate(
    sports_balls = round(sports_balls),
    sports_balls = as.integer(sports_balls)
  )
```

Read the percipitation data for 2018 and 2017

``` r
precip_2018 = 
  read_excel(
    "./data/Trash-Wheel-Collection-Totals-8-6-19.xlsx", 
    sheet = "2018 Precipitation", 
    skip = 1) %>% 
  janitor::clean_names() %>% 
  drop_na(month) %>% 
  mutate(year = 2018) %>% 
  relocate(year)
  
precip_2017 = 
  read_excel(
    "./data/Trash-Wheel-Collection-Totals-8-6-19.xlsx", 
    sheet = "2017 Precipitation", 
    skip = 1) %>% 
  janitor::clean_names() %>% 
  drop_na(month) %>% 
  mutate(year = 2017) %>% 
  relocate(year)
```

Now combine annual precipitation

``` r
month_df =
  tibble(
    month = 1:12,
    month_name = month.name
  )

precip_df =
  bind_rows(precip_2018, precip_2017)

left_join(precip_df, month_df, by = "month")
```

    ## # A tibble: 24 x 4
    ##     year month total month_name
    ##    <dbl> <dbl> <dbl> <chr>     
    ##  1  2018     1  0.94 January   
    ##  2  2018     2  4.8  February  
    ##  3  2018     3  2.69 March     
    ##  4  2018     4  4.69 April     
    ##  5  2018     5  9.27 May       
    ##  6  2018     6  4.77 June      
    ##  7  2018     7 10.2  July      
    ##  8  2018     8  6.45 August    
    ##  9  2018     9 10.5  September 
    ## 10  2018    10  2.12 October   
    ## # … with 14 more rows

This dataset contains information from the Mr. Trashwheel trash colector
in Baltimore, MD. As trash enters the inner harbor, the trashwheel
collects that trash, and stores it in a dumpster. The dataset contains
information on year, month, and trash collected, including guitars.
There are a total of 344 rows in our final dataset. Additional data
sheets include month precipitation data.

  - The median number of sports balls found in a dumpster in 2017 was 8
  - The total precipitation in 2018 was 70.33 inches.

## Problem 2

Read, clean, and tidy the NYC transit dataset

``` r
transit_df = 
  read_csv(
    "./data/NYC_Transit_Subway_Entrance_And_Exit_Data.csv") %>% 
  janitor::clean_names() %>% 
  select(line, station_name, station_latitude, station_longitude, route1:route11, entrance_type, vending, entry, ada) %>% 
  mutate(
    route8 = as.character(route8),
    route9 = as.character(route9),
    route10 = as.character(route10),
    route11 = as.character(route11)
  ) %>% 
  mutate(
    entry = as.logical(recode(entry, "YES" = "TRUE", "NO" = "FALSE"))
  )
```

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_character(),
    ##   `Station Latitude` = col_double(),
    ##   `Station Longitude` = col_double(),
    ##   Route8 = col_double(),
    ##   Route9 = col_double(),
    ##   Route10 = col_double(),
    ##   Route11 = col_double(),
    ##   ADA = col_logical(),
    ##   `Free Crossover` = col_logical(),
    ##   `Entrance Latitude` = col_double(),
    ##   `Entrance Longitude` = col_double()
    ## )

    ## See spec(...) for full column specifications.

``` r
transit_df_tidy =
  transit_df %>% 
  pivot_longer(
    route1:route11,
    names_to = "route",
    names_prefix = "route",
    values_to = "route_name"
  ) %>% 
  drop_na(route_name) %>% 
  distinct(line, station_name, route, route_name, .keep_all = TRUE) %>% 
  relocate(line, station_name, route, route_name)
```

The NYC transit dataset contains geographical information about stations
and lines in NYC and the routes for train lines associated with them.
There are 36 lines, specifically (4 Avenue, 42nd St Shuttle, 6 Avenue,
63rd Street, 8 Avenue, Archer Av, Astoria, Brighton, Broadway, Broadway
Jamaica, Broadway-7th Ave, Canarsie, Clark, Concourse, Coney Island,
Crosstown, Culver, Dyre Av, Eastern Parkway, Flushing, Franklin, Fulton,
Jerome, Lenox, Lexington, Liberty, Myrtle, Nassau, New Lots, Nostrand,
Pelham, Queens Boulevard, Rockaway, Sea Beach, West End, White Plains
Road). The initial table was not tidy. The second dataframe I created,
transit\_df\_tidy, is tidy because its in the longer format.

  - The total number of stations are 259.

  - The total number of stations that are ADA compliant are 167.

  - The proportion of station entrances / exits without vending allow
    entrance is 0.82.

  - The A train is served by 12 stations.

  - Of those stations, 7 are ADA compliant.

## Problem 3

Read and load pols-month, snp.csv, and unemployment datasets

``` r
month_df =
  tibble(
    month = 1:12,
    month_name = month.name
  )

pols_month_df =
  read_csv("./data/fivethirtyeight_datasets/pols-month.csv") %>% 
  janitor::clean_names() %>% 
  separate(mon, into = c("year", "month", "day"), sep = "-") %>% 
  mutate(
    month = as.integer(month)
  ) %>% 
  left_join(month_df, by = "month") %>% 
  relocate(month_name, .after = year) %>% 
  mutate(
    president = if_else(prez_gop == 1, "GOP", "DEM")
  ) %>% 
  select(-month, -day, -prez_gop, -prez_dem) %>% 
  arrange(year, month_name)
```

    ## Parsed with column specification:
    ## cols(
    ##   mon = col_date(format = ""),
    ##   prez_gop = col_double(),
    ##   gov_gop = col_double(),
    ##   sen_gop = col_double(),
    ##   rep_gop = col_double(),
    ##   prez_dem = col_double(),
    ##   gov_dem = col_double(),
    ##   sen_dem = col_double(),
    ##   rep_dem = col_double()
    ## )

``` r
snp_df =
  read_csv("./data/fivethirtyeight_datasets/snp.csv") %>% 
  janitor::clean_names() %>% 
  separate(date, into = c("month", "day", "year"), sep = "/")%>% 
  mutate(
    month = as.integer(month)
  ) %>% 
  left_join(month_df, by = "month") %>% 
  relocate(month_name, .after = year) %>% 
  select(-month, -day) %>% 
  arrange(year, month_name)
```

    ## Parsed with column specification:
    ## cols(
    ##   date = col_character(),
    ##   close = col_double()
    ## )

``` r
month_df =
  tibble(
    month_name = month.name,
    month = tolower(month.abb)
  )

unemployment_df =
  read_csv("./data/fivethirtyeight_datasets/unemployment.csv") %>% 
  janitor::clean_names()
```

    ## Parsed with column specification:
    ## cols(
    ##   Year = col_double(),
    ##   Jan = col_double(),
    ##   Feb = col_double(),
    ##   Mar = col_double(),
    ##   Apr = col_double(),
    ##   May = col_double(),
    ##   Jun = col_double(),
    ##   Jul = col_double(),
    ##   Aug = col_double(),
    ##   Sep = col_double(),
    ##   Oct = col_double(),
    ##   Nov = col_double(),
    ##   Dec = col_double()
    ## )

``` r
unemployment_df_tidy =
  unemployment_df %>% 
  pivot_longer(
    jan:dec,
    names_to = "month",
    values_to = "unemployment_rate"
  ) %>% 
  left_join(month_df, by = "month") %>% 
  select(-month) %>% 
  arrange(year, month_name) %>% 
  mutate(
    year = as.character(year)
  )
```

Now, join the data.

``` r
joined_df =
  left_join(pols_month_df, snp_df, by = c("year", "month_name")) %>% 
  left_join(unemployment_df_tidy, by = c("year", "month_name"))
```

The pols dataset contained what the breakdown of GOP vs DEM presidents
were and the GOP vs DEM breakdown amongst house representatives,
senators, and state governors. The snp dataset was the S\&P 500 market
price per month that every year. The unemployment dataset was the
unemployment rate associated with every year. There’s a total of 822
observations in the joined dataset over the course of 1947 through 2015.
