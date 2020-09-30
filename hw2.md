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
