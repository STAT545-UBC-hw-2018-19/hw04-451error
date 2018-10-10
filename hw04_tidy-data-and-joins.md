hm04
================

## Load packages

``` r
library(gapminder)
library(tidyverse)
library(knitr)
```

# Cheatsheet for dplyr join functions

## The data

I used the data found
[here](https://www.kaggle.com/unsdsn/world-happiness/). This is The
World Happiness Report is a survey of scores based on data from the
Gallup World Poll. More details about the dataset can be found
[here](https://www.kaggle.com/unsdsn/world-happiness/home)

``` r
happy_2016 = read_csv("2016.csv")
```

    ## Parsed with column specification:
    ## cols(
    ##   Country = col_character(),
    ##   Region = col_character(),
    ##   `Happiness Rank` = col_integer(),
    ##   `Happiness Score` = col_double(),
    ##   `Lower Confidence Interval` = col_double(),
    ##   `Upper Confidence Interval` = col_double(),
    ##   `Economy (GDP per Capita)` = col_double(),
    ##   Family = col_double(),
    ##   `Health (Life Expectancy)` = col_double(),
    ##   Freedom = col_double(),
    ##   `Trust (Government Corruption)` = col_double(),
    ##   Generosity = col_double(),
    ##   `Dystopia Residual` = col_double()
    ## )

``` r
happy_2017 = read_csv("2017.csv")
```

    ## Parsed with column specification:
    ## cols(
    ##   Country = col_character(),
    ##   Happiness.Rank = col_integer(),
    ##   Happiness.Score = col_double(),
    ##   Whisker.high = col_double(),
    ##   Whisker.low = col_double(),
    ##   Economy..GDP.per.Capita. = col_double(),
    ##   Family = col_double(),
    ##   Health..Life.Expectancy. = col_double(),
    ##   Freedom = col_double(),
    ##   Generosity = col_double(),
    ##   Trust..Government.Corruption. = col_double(),
    ##   Dystopia.Residual = col_double()
    ## )

``` r
names(happy_2016)
```

    ##  [1] "Country"                       "Region"                       
    ##  [3] "Happiness Rank"                "Happiness Score"              
    ##  [5] "Lower Confidence Interval"     "Upper Confidence Interval"    
    ##  [7] "Economy (GDP per Capita)"      "Family"                       
    ##  [9] "Health (Life Expectancy)"      "Freedom"                      
    ## [11] "Trust (Government Corruption)" "Generosity"                   
    ## [13] "Dystopia Residual"

``` r
names(happy_2017)
```

    ##  [1] "Country"                       "Happiness.Rank"               
    ##  [3] "Happiness.Score"               "Whisker.high"                 
    ##  [5] "Whisker.low"                   "Economy..GDP.per.Capita."     
    ##  [7] "Family"                        "Health..Life.Expectancy."     
    ##  [9] "Freedom"                       "Generosity"                   
    ## [11] "Trust..Government.Corruption." "Dystopia.Residual"

We can see that there are tables where the corresponding columns are
named the same, so we first update the column names:

``` r
happy_2017 = happy_2017 %>% 
  rename(`Happiness Rank`=`Happiness.Rank`,
         `Happiness Score`=`Happiness.Score`,
         `Upper Confidence Interval`=`Whisker.high`,
         `Lower Confidence Interval`=`Whisker.low`,
         `Health (Life Expectancy)`=`Health..Life.Expectancy.`,
         `Economy (GDP per Capita)` = `Economy..GDP.per.Capita.`,
         `Trust (Government Corruption)`=`Trust..Government.Corruption.`,
         `Dystopia Residual`=`Dystopia.Residual`)
```

\#left\_join

We see that `happy_2017` does not have a “Region” variable. We can
create a “Region” table from the `happy_2016` data and use `left_join`
to pull in the “Region” in the 2017 data:

``` r
region = happy_2016 %>% 
  select(Country,Region)

(happy_2017_with_region = happy_2017 %>% 
  left_join(region,by="Country"))
```

    ## # A tibble: 155 x 13
    ##    Country `Happiness Rank` `Happiness Scor~ `Upper Confiden~
    ##    <chr>              <int>            <dbl>            <dbl>
    ##  1 Norway                 1             7.54             7.59
    ##  2 Denmark                2             7.52             7.58
    ##  3 Iceland                3             7.50             7.62
    ##  4 Switze~                4             7.49             7.56
    ##  5 Finland                5             7.47             7.53
    ##  6 Nether~                6             7.38             7.43
    ##  7 Canada                 7             7.32             7.38
    ##  8 New Ze~                8             7.31             7.38
    ##  9 Sweden                 9             7.28             7.34
    ## 10 Austra~               10             7.28             7.36
    ## # ... with 145 more rows, and 9 more variables: `Lower Confidence
    ## #   Interval` <dbl>, `Economy (GDP per Capita)` <dbl>, Family <dbl>,
    ## #   `Health (Life Expectancy)` <dbl>, Freedom <dbl>, Generosity <dbl>,
    ## #   `Trust (Government Corruption)` <dbl>, `Dystopia Residual` <dbl>,
    ## #   Region <chr>

## inner\_join

We can try to use `inner_join` on the same two tables but this will give
a different result:

``` r
happy_2017 %>% 
  inner_join(region,by="Country")
```

    ## # A tibble: 150 x 13
    ##    Country `Happiness Rank` `Happiness Scor~ `Upper Confiden~
    ##    <chr>              <int>            <dbl>            <dbl>
    ##  1 Norway                 1             7.54             7.59
    ##  2 Denmark                2             7.52             7.58
    ##  3 Iceland                3             7.50             7.62
    ##  4 Switze~                4             7.49             7.56
    ##  5 Finland                5             7.47             7.53
    ##  6 Nether~                6             7.38             7.43
    ##  7 Canada                 7             7.32             7.38
    ##  8 New Ze~                8             7.31             7.38
    ##  9 Sweden                 9             7.28             7.34
    ## 10 Austra~               10             7.28             7.36
    ## # ... with 140 more rows, and 9 more variables: `Lower Confidence
    ## #   Interval` <dbl>, `Economy (GDP per Capita)` <dbl>, Family <dbl>,
    ## #   `Health (Life Expectancy)` <dbl>, Freedom <dbl>, Generosity <dbl>,
    ## #   `Trust (Government Corruption)` <dbl>, `Dystopia Residual` <dbl>,
    ## #   Region <chr>

We see that we lost a few results. There are countries in `region` not
on `happy_2017` and countries in `happy_2017` that are not in `region`;
`inner_join` removes all these results whereas `left_join` kept all
columns from `happy_2017`. We can see which countries in `happy_2017`
that were not in `region` using masking:

``` r
happy_2017_with_region[is.na(happy_2017_with_region$Region),]
```

    ## # A tibble: 5 x 13
    ##   Country `Happiness Rank` `Happiness Scor~ `Upper Confiden~
    ##   <chr>              <int>            <dbl>            <dbl>
    ## 1 Taiwan~               33             6.42             6.49
    ## 2 Hong K~               71             5.47             5.55
    ## 3 Mozamb~              113             4.55             4.77
    ## 4 Lesotho              139             3.81             4.04
    ## 5 Centra~              155             2.69             2.86
    ## # ... with 9 more variables: `Lower Confidence Interval` <dbl>, `Economy
    ## #   (GDP per Capita)` <dbl>, Family <dbl>, `Health (Life
    ## #   Expectancy)` <dbl>, Freedom <dbl>, Generosity <dbl>, `Trust
    ## #   (Government Corruption)` <dbl>, `Dystopia Residual` <dbl>,
    ## #   Region <chr>

However, there is also a simpler way of getting this result using
`dplyr` as can be seen in the next section.

## anti\_join

> `anti_join(x,y)` returns all rows from `x` where there are not
> matching values in `y`, keeping just columns from `x`

We can use `anti_join` to find the countries in `happy_2017` but not in
`region`:

``` r
happy_2017 %>% 
  anti_join(region,by='Country')
```

    ## # A tibble: 5 x 12
    ##   Country `Happiness Rank` `Happiness Scor~ `Upper Confiden~
    ##   <chr>              <int>            <dbl>            <dbl>
    ## 1 Taiwan~               33             6.42             6.49
    ## 2 Hong K~               71             5.47             5.55
    ## 3 Mozamb~              113             4.55             4.77
    ## 4 Lesotho              139             3.81             4.04
    ## 5 Centra~              155             2.69             2.86
    ## # ... with 8 more variables: `Lower Confidence Interval` <dbl>, `Economy
    ## #   (GDP per Capita)` <dbl>, Family <dbl>, `Health (Life
    ## #   Expectancy)` <dbl>, Freedom <dbl>, Generosity <dbl>, `Trust
    ## #   (Government Corruption)` <dbl>, `Dystopia Residual` <dbl>

Similarly, we can find the countries in `region` but not in
`happy_2017`:

``` r
region %>% 
  anti_join(happy_2017,by="Country")
```

    ## # A tibble: 7 x 2
    ##   Country           Region                     
    ##   <chr>             <chr>                      
    ## 1 Puerto Rico       Latin America and Caribbean
    ## 2 Taiwan            Eastern Asia               
    ## 3 Suriname          Latin America and Caribbean
    ## 4 Hong Kong         Eastern Asia               
    ## 5 Somaliland Region Sub-Saharan Africa         
    ## 6 Laos              Southeastern Asia          
    ## 7 Comoros           Sub-Saharan Africa

From this, we see that there are entries in the Country column that were
not consistently named from previous years, such as Taiwan and Hong
Kong. We can update this using
`if_else`:

``` r
happy_2017$Country = if_else(happy_2017$Country == "Taiwan Province of China","Taiwan",happy_2017$Country)

happy_2017$Country = if_else(happy_2017$Country == "Hong Kong S.A.R., China","Hong Kong",happy_2017$Country)
```

After this update, we should have fewer entries when we call
`anti_join`:

``` r
happy_2017 %>% 
  anti_join(region,by='Country')
```

    ## # A tibble: 3 x 12
    ##   Country `Happiness Rank` `Happiness Scor~ `Upper Confiden~
    ##   <chr>              <int>            <dbl>            <dbl>
    ## 1 Mozamb~              113             4.55             4.77
    ## 2 Lesotho              139             3.81             4.04
    ## 3 Centra~              155             2.69             2.86
    ## # ... with 8 more variables: `Lower Confidence Interval` <dbl>, `Economy
    ## #   (GDP per Capita)` <dbl>, Family <dbl>, `Health (Life
    ## #   Expectancy)` <dbl>, Freedom <dbl>, Generosity <dbl>, `Trust
    ## #   (Government Corruption)` <dbl>, `Dystopia Residual` <dbl>

## semi\_join

`semi_join` is similar to `left_join`, but where `left_join(x,y)` will
return a row from `x` for each match in `y`, `semi_join(x,y)` will only
return one row of `x`. `semi_join` also filters out rows in `x` where
there is no match in `y`

``` r
happy_2017 %>% 
  semi_join(region,by="Country")
```

    ## # A tibble: 152 x 12
    ##    Country `Happiness Rank` `Happiness Scor~ `Upper Confiden~
    ##    <chr>              <int>            <dbl>            <dbl>
    ##  1 Norway                 1             7.54             7.59
    ##  2 Denmark                2             7.52             7.58
    ##  3 Iceland                3             7.50             7.62
    ##  4 Switze~                4             7.49             7.56
    ##  5 Finland                5             7.47             7.53
    ##  6 Nether~                6             7.38             7.43
    ##  7 Canada                 7             7.32             7.38
    ##  8 New Ze~                8             7.31             7.38
    ##  9 Sweden                 9             7.28             7.34
    ## 10 Austra~               10             7.28             7.36
    ## # ... with 142 more rows, and 8 more variables: `Lower Confidence
    ## #   Interval` <dbl>, `Economy (GDP per Capita)` <dbl>, Family <dbl>,
    ## #   `Health (Life Expectancy)` <dbl>, Freedom <dbl>, Generosity <dbl>,
    ## #   `Trust (Government Corruption)` <dbl>, `Dystopia Residual` <dbl>

We see that fewer results than when we used `left_join`. We can see the
countries that are missing are Mozambique, Lesotho, and Central African
Republic, which correspond to the last `anti_join` call.

## full\_join

If we want to keep all the countries in both datasets, we would use
`full_join`:

``` r
full_join(happy_2017,region)
```

    ## Joining, by = "Country"

    ## # A tibble: 160 x 13
    ##    Country `Happiness Rank` `Happiness Scor~ `Upper Confiden~
    ##    <chr>              <int>            <dbl>            <dbl>
    ##  1 Norway                 1             7.54             7.59
    ##  2 Denmark                2             7.52             7.58
    ##  3 Iceland                3             7.50             7.62
    ##  4 Switze~                4             7.49             7.56
    ##  5 Finland                5             7.47             7.53
    ##  6 Nether~                6             7.38             7.43
    ##  7 Canada                 7             7.32             7.38
    ##  8 New Ze~                8             7.31             7.38
    ##  9 Sweden                 9             7.28             7.34
    ## 10 Austra~               10             7.28             7.36
    ## # ... with 150 more rows, and 9 more variables: `Lower Confidence
    ## #   Interval` <dbl>, `Economy (GDP per Capita)` <dbl>, Family <dbl>,
    ## #   `Health (Life Expectancy)` <dbl>, Freedom <dbl>, Generosity <dbl>,
    ## #   `Trust (Government Corruption)` <dbl>, `Dystopia Residual` <dbl>,
    ## #   Region <chr>

Where there are no matching values, `NA` is filled in the corresponding
columns.

# Data Reshaping cheatsheet

I will use the same dataset that was imported above.

## filter

Filter is used to select desired rows in our data based on specified
criteria. Let’s look at the top 10 happiest countries in 2017 based on
the happiness index using `filter`:

``` r
happy_2017 %>% 
  filter(`Happiness Rank` <= 10)
```

    ## # A tibble: 10 x 12
    ##    Country `Happiness Rank` `Happiness Scor~ `Upper Confiden~
    ##    <chr>              <int>            <dbl>            <dbl>
    ##  1 Norway                 1             7.54             7.59
    ##  2 Denmark                2             7.52             7.58
    ##  3 Iceland                3             7.50             7.62
    ##  4 Switze~                4             7.49             7.56
    ##  5 Finland                5             7.47             7.53
    ##  6 Nether~                6             7.38             7.43
    ##  7 Canada                 7             7.32             7.38
    ##  8 New Ze~                8             7.31             7.38
    ##  9 Sweden                 9             7.28             7.34
    ## 10 Austra~               10             7.28             7.36
    ## # ... with 8 more variables: `Lower Confidence Interval` <dbl>, `Economy
    ## #   (GDP per Capita)` <dbl>, Family <dbl>, `Health (Life
    ## #   Expectancy)` <dbl>, Freedom <dbl>, Generosity <dbl>, `Trust
    ## #   (Government Corruption)` <dbl>, `Dystopia Residual` <dbl>

## select()

Select is used to choose specific columns of our dataset. This is a
contrast from `filter` as `filter` chooses rows.

``` r
happysubset = happy_2017 %>%
  mutate(year=rep(2017,nrow(happy_2017))) %>% 
  bind_rows(happy_2016 %>% mutate(year=rep(2016,nrow(happy_2016)))) %>% 
  select(Country,`Happiness Rank`,year)
```

## spread

Spread takes values from a column and converts it into multiple columns.

``` r
happysubset = happysubset %>% 
  left_join(region,by="Country")

wide_happysubset = happysubset %>% 
  spread(key = 'year', value = `Happiness Rank`)
wide_happysubset
```

    ## # A tibble: 160 x 4
    ##    Country     Region                          `2016` `2017`
    ##    <chr>       <chr>                            <int>  <int>
    ##  1 Afghanistan Southern Asia                      154    141
    ##  2 Albania     Central and Eastern Europe         109    109
    ##  3 Algeria     Middle East and Northern Africa     38     53
    ##  4 Angola      Sub-Saharan Africa                 141    140
    ##  5 Argentina   Latin America and Caribbean         26     24
    ##  6 Armenia     Central and Eastern Europe         121    121
    ##  7 Australia   Australia and New Zealand            9     10
    ##  8 Austria     Western Europe                      12     13
    ##  9 Azerbaijan  Central and Eastern Europe          81     85
    ## 10 Bahrain     Middle East and Northern Africa     42     41
    ## # ... with 150 more rows

## gather

Given a data in a wide form, we would like to put it in a tidy form,
where each column represents a variable and each row contains the value
of the variable. We use the `wide_happysubset` defined above to
illustrate this function:

``` r
wide_happysubset %>% 
  gather(key = 'Year',value = `Happiness Rank`,3:4)
```

    ## # A tibble: 320 x 4
    ##    Country     Region                          Year  `Happiness Rank`
    ##    <chr>       <chr>                           <chr>            <int>
    ##  1 Afghanistan Southern Asia                   2016               154
    ##  2 Albania     Central and Eastern Europe      2016               109
    ##  3 Algeria     Middle East and Northern Africa 2016                38
    ##  4 Angola      Sub-Saharan Africa              2016               141
    ##  5 Argentina   Latin America and Caribbean     2016                26
    ##  6 Armenia     Central and Eastern Europe      2016               121
    ##  7 Australia   Australia and New Zealand       2016                 9
    ##  8 Austria     Western Europe                  2016                12
    ##  9 Azerbaijan  Central and Eastern Europe      2016                81
    ## 10 Bahrain     Middle East and Northern Africa 2016                42
    ## # ... with 310 more rows
