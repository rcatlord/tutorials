tasteR
================

This is just a taster of some of the capabililies of [R](https://www.r-project.org/). It is not intended to cover all of the fundamentals of R. Instead, we'll run through the process of importing, tidying, transforming, visualising and reporting data using some of the packages developed by [Hadley Wickham](http://hadley.nz/) and others. These packages help lower the barrier to entry for newbies and will hopefully inspire you to code regularly in R.

#### Learning outcomes

By the end of the session participants will:

-   Gain a working knowledge of [RStudio](https://www.rstudio.com/) software
-   Become familiar with the process of importing, tidying, transforming, querying, visualising and reporting data
-   Create and publish a data visualisation

### Introduction

#### What is R?

R is an open source programming language for statistical analysis and data visualisation. It was developed by Ross Ihaka and Robert Gentleman of the University of Auckland and released in 1995 [(see this NYT piece from 2009)](http://www.nytimes.com/2009/01/07/technology/business-computing/07program.html?pagewanted=all&_r=0). R is widely used in [academia](http://www.nature.com/news/programming-tools-adventures-with-r-1.16609) and becoming increasingly important in business and [government](https://data.blog.gov.uk/2015/06/12/getting-started-with-data-science/).

#### Why use R?

-   *It's the leading tool for statistical analysis, forecasting and machine learning*
-   *Cutting edge analytics*: over 8,000 user-contributed packages available on finance, genomics, animal tracking, crime analysis, and much more
-   *Powerful graphics and data visualisations*: used by the New York Times and FiveThirtyEight
-   *Open source*: no vendor lock-in
-   *Reproducibility*: code can be shared and the results repeated
-   *Transparency*: explicitly documents all the steps of your analyses
-   *Automation*: analyses can be run and re-run with new and existing datasets
-   *Support network*: worldwide community of developers and users

#### Are there any disadvantages?

Learning R can be a steep learning curve and the transition from a graphical user interface like Excel or SPSS to one that is command driven can be unsettling. However, you’ll soon find that working with a command line is much more efficient than pointing and clicking. After all, you can replicate, automate and share your R scripts.

### Setting up

#### Installing R and RStudio

Download and install [R](https://cran.r-project.org/) and [RStudio](https://www.rstudio.com/). RStudio is an integrated development environment for R which includes syntax highlighting, code completion, debugging tools and an in-built browser. RStudio makes R a much more user-friendly experience.

#### Packages

Packages are collections of R functions and data. There are over 8,000 user-contributed packages available to install from [CRAN](https://cran.r-project.org/web/packages/). Just type `install.packages()` in the console with the name of the package in inverted commas. When you install a package in R for the first time you will be asked to select a CRAN mirror. A mirror is a distribution site for R source code, manuals, and contributed packages. Just pick the mirror that is closest to you.

The code below will install the [dplyr](https://cran.r-project.org/web/packages/dplyr/index.html) package which is a set of tools for manipulating dataframes. It will also install all its dependent packages.

``` r
install.packages("dplyr", dependencies = TRUE)
```

Once installed the package can be loaded into your R session with the `library()` function.

``` r
library(dplyr)
```

A helpful list of R packages hand-picked by RStudio is available at this link: <https://github.com/rstudio/RStartHere>

### Useful tips

-   If you need information on a function just type `?` or `help()`, e.g. `?mean` or `help(mean)`.
-   If you have a more complicated question try trawling through the answers on [stackoverflow.com](stackoverflow.com) with \[r\] in the search field.
-   The `#` symbol can be used to add comments to your code. R will ignore everything after the `#` symbol.
-   Every package on CRAN comes with a vignette which can be loaded by typing `browseVignettes(package = "")` with the name of the package entered in inverted commas.
-   There are a couple of useful style guides from [Google](Google's%20R%20Style%20Guide) and [Hadley Wickham](http://adv-r.had.co.nz/Style.html).

### Importing data

R can handle a range of data formats: .xlsx, .csv, .txt, .sav, .shp etc. Some data formats require specific packages.

To import a .csv file you can use the function `read_csv()` from the [readr](https://cran.r-project.org/web/packages/readr/index.html) package.

``` r
library(readr)
df <- read_csv("world_prison_population_list_11th_edition_wide.csv")
```

The data that we've imported derive the World Prison Population List which is published by the [International Centre for Prison Studies](http://www.prisonstudies.org/). The most recent report, the [11th edition](http://www.prisonstudies.org/sites/default/files/resources/downloads/world_prison_population_list_11th_edition.pdf), uses data from 223 countries and is accurate up to October 2015.

Let's have a look at the first few columns of data using the `select()` function in dplyr. The `%>%` operator 'chains' lines of code together and can be read as 'then'. So, first we call the dataframe 'df' and then select the first 3 columns.

``` r
df %>% select(1:3)
```

    ## # A tibble: 4 × 3
    ##                            Name        Afghanistan         Albania
    ##                           <chr>              <chr>           <chr>
    ## 1                          iso3                AFG             ALB
    ## 2                        Region South Central Asia Southern Europe
    ## 3 Estimated national population           35600000         2890000
    ## 4       Prison population total              26519            5455

You'll notice that the data structure is messy<sup>[\*](#myfootnote1)</sup>. Variables are stored in rows and values are column headers. This type of tabular data structure is very common but not particularly helpful for data analysis.

### Tidying data

According to Hadley Wickham, *tidy data* are structured for use in R and satisfy three rules [(Grolemund and Wickham 2016)](http://r4ds.had.co.nz/tidy-data.html):

-   variables are stored upright in columns;
-   observations are lined up in rows, and;
-   values are placed in their own cells.

Assigning variables to columns ensures that values are paired with other values in the same row of observations.

The [tidyr](https://cran.r-project.org/web/packages/tidyr/index.html) package has helpful tools called `gather()` and `spread()` which will fix this for us.

The code below gathers all of the columns except the first and then spreads the gathered columns.

``` r
library(tidyr)
```

    ## Warning: package 'tidyr' was built under R version 3.3.2

``` r
df <- df %>%
  gather(country, value, 2:ncol(df)) %>%
  spread(Name, value)
```

The function `glimpse()` from the dplyr package prints out the variables from the dataframe and the first few rows. Additional information on data types is also provided. Our data is now structured tidily but the variable names are too long, in the wrong order, 'region' needs to be a factor variable, and the population values should be integers not characters.

``` r
glimpse(df) 
```

    ## Observations: 221
    ## Variables: 5
    ## $ country                       <chr> "Afghanistan", "Albania", "Alger...
    ## $ Estimated national population <chr> "35600000", "2890000", "37280000...
    ## $ iso3                          <chr> "AFG", "ALB", "DZA", "WSM", "AND...
    ## $ Prison population total       <chr> "26519", "5455", "60220", "214",...
    ## $ Region                        <chr> "South Central Asia", "Southern ...

We'll use the `select()` and `mutate()` functions from the dplyr package to remedy this. The variables are selected and renamed using the `select()` function and re-classed using `mutate()`.

``` r
df <- df %>% 
  select(country,
         iso3,
         region = Region,
         national_pop = `Estimated national population`,
         prison_pop = `Prison population total`) %>% 
  mutate(region = factor(region),
         national_pop = as.integer(national_pop),
         prison_pop = as.integer(prison_pop))
glimpse(df)
```

    ## Observations: 221
    ## Variables: 5
    ## $ country      <chr> "Afghanistan", "Albania", "Algeria", "American Sa...
    ## $ iso3         <chr> "AFG", "ALB", "DZA", "WSM", "AND", "AGO", "AIA", ...
    ## $ region       <fctr> South Central Asia, Southern Europe, Northern Af...
    ## $ national_pop <int> 35600000, 2890000, 37280000, 56000, 76250, 214500...
    ## $ prison_pop   <int> 26519, 5455, 60220, 214, 55, 22826, 46, 343, 6906...

### Transforming data

Creating new variables by transforming data is straightforward. The following code uses the `mutate()` function in dplyr to create a new variable representing the rate of incarceration per 100,000 people. The resulting values are rounded using the base R `round()` function.

``` r
df <- mutate(df, rate = round((prison_pop / national_pop) * 100000, 0))
```

Let's use the `slice()` function to view the first 5 rows of the 'rate' variable.

``` r
df %>% slice(1:5) %>% select(rate)
```

    ## # A tibble: 5 × 1
    ##    rate
    ##   <dbl>
    ## 1    74
    ## 2   189
    ## 3   162
    ## 4   382
    ## 5    72

### Querying data

Arranging values is possible using the `arrange()` function in dplyr. The code below sorts the 'rate' values in ascending order and then prints the first 5 rows.

``` r
arrange(df, rate) %>% head(5)
```

    ## # A tibble: 5 × 6
    ##             country  iso3          region national_pop prison_pop  rate
    ##               <chr> <chr>          <fctr>        <int>      <int> <dbl>
    ## 1     Guinea Bissau   GNB  Western Africa      1725000         92     5
    ## 2        San Marino   SMR Southern Europe        33700          2     6
    ## 3     Liechtenstein   LIE  Western Europe        37370          8    21
    ## 4 Faeroes (Denmark)   FRO Northern Europe        48480         11    23
    ## 5  Guinea (Rep. of)   GUY  Western Africa     12120000       3110    26

In descending order:

``` r
arrange(df, desc(rate)) %>% head(5)
```

    ## # A tibble: 5 × 6
    ##            country  iso3         region national_pop prison_pop  rate
    ##              <chr> <chr>         <fctr>        <int>      <int> <dbl>
    ## 1       Seychelles   SYC Eastern Africa        92000        735   799
    ## 2           U.S.A.   USA  North America    317760000    2217000   698
    ## 3 St Kitts & Nevis   KNA      Caribbean        55000        334   607
    ## 4     Turkmenistan   TKM   Central Asia      5240000      30568   583
    ## 5 Virgin Is. (USA)   VIR      Caribbean       106700        577   541

Then sorted by 'region' (ascending) and then by 'rate' in descending order

``` r
arrange(df, region, desc(rate)) %>% head(5)
```

    ## # A tibble: 5 × 6
    ##            country  iso3    region national_pop prison_pop  rate
    ##              <chr> <chr>    <fctr>        <int>      <int> <dbl>
    ## 1 St Kitts & Nevis   KNA Caribbean        55000        334   607
    ## 2 Virgin Is. (USA)   VIR Caribbean       106700        577   541
    ## 3             Cuba   CUB Caribbean     11250000      57337   510
    ## 4  Virgin Is. (UK)   VGB Caribbean        28000        119   425
    ## 5          Grenada   GRD Caribbean       106500        424   398

The `group_by()` function allows you to run operations on groups of data. The following code groups the data by 'region', calculates the total prison population, and then sorts the results in decscending order.

``` r
df %>% 
  group_by(region) %>% 
  summarise(total = sum(prison_pop)) %>% 
  arrange(desc(total))
```

    ## # A tibble: 20 × 2
    ##                        region   total
    ##                        <fctr>   <int>
    ## 1               North America 2255210
    ## 2                Eastern Asia 1850147
    ## 3               South America 1036812
    ## 4          South Eastern Asia  881634
    ## 5          South Central Asia  859473
    ## 6                 Europe/Asia  851674
    ## 7              Eastern Africa  395974
    ## 8             Central America  368027
    ## 9  Central and Eastern Europe  266752
    ## 10            Northern Africa  247194
    ## 11            Southern Europe  172817
    ## 12            Southern Africa  172316
    ## 13               Western Asia  171696
    ## 14             Western Europe  163674
    ## 15               Central Asia  134847
    ## 16             Western Africa  133753
    ## 17                  Caribbean  120479
    ## 18             Central Africa   89498
    ## 19            Northern Europe   80381
    ## 20                    Oceania   54726

### Exercises

Try and find the answers to these questions by using the data wrangling tools provided by dplyr:

1.  How many people are held in penal institutions worldwide?

2.  What is the prison population of the U.S.A?

3.  Which country has the highest prison population rate?

4.  Which country has the second highest prison population rate?

5.  Which country has the lowest prison population rate?

6.  What is the world prison population rate?

7.  What is the median incarceration rate for Oceania?

8.  If the U.S.A. has 4% of the world's population what percent of the world's prison population does it have?

### Visualising data

There are several packages that will allow you to visualise data both statically and interactively. Two of the most popular packages are [ggplot2](https://cran.r-project.org/web/packages/ggplot2/index.html) for static outputs and [highcharter](https://cran.r-project.org/web/packages/highcharter/index.html) which is an R wrapper for the [Highcharts](http://www.highcharts.com/) javascript libray.

#### Static plots

First, we'll attempt to create a static plot in ggplot2 by subsetting the top 10 countries by incarceration rate.

``` r
temp <- df %>% 
  arrange(desc(rate)) %>% 
  slice(1:10) 
```

Then we load the ggplot2 library and build up our plot.

``` r
library(ggplot2) ; library(ggthemes)
```

    ## Warning: package 'ggthemes' was built under R version 3.3.2

``` r
ggplot(temp, aes(reorder(country, rate), rate))+
  theme_tufte(base_size=14, ticks=F) +
  geom_bar(width=0.25, fill="gray", stat="identity") +
  theme(axis.title=element_blank()) +
  scale_y_continuous(breaks=seq(0, 800, 100)) + 
  geom_hline(yintercept=seq(0, 800, 100), col="white", lwd=1) +
  labs(x="", y="\nRate per 100,000 population") +
  coord_flip() +
  ggtitle("Top 10 countries by incarceration rate") 
```

![](https://github.com/rcatlord/tutorials/blob/master/tasteR/plot.png)

You'll notice that we also loaded the [ggthemes](https://cran.r-project.org/web/packages/ggthemes/index.html) package. The `theme_tufte()` function allowed us to style the plot in a manner similar to those adopted in [Edward Tufte's](https://www.edwardtufte.com/tufte/) graphics.

You can save the plot with the `ggsave()` function.

``` r
ggsave("plot.png", scale = 1, dpi = 300)
```

<br>

#### Interactive plots

The next plot is interactive and uses the highcharter package. The theme adopts the in-house style of [fivethirtyeight.com](http://fivethirtyeight.com/).

``` r
library(highcharter)

hc <- highchart(height = 400, width = 700) %>%
  hc_title(text = "Top 10 countries by incarceration rate") %>% 
  hc_subtitle(text = "Source: International Centre for Prison Studies") %>% 
  hc_xAxis(categories = temp$country) %>% 
  hc_add_series(name = "Incarceration rate", data = temp$rate, 
                type = 'bar', color = "#f16913") %>% 
  hc_yAxis(title = list(text = "Rate per 100,000 population")) %>% 
  hc_legend(enabled = FALSE) %>% 
  hc_exporting(enabled = TRUE)
hc %>% hc_add_theme(hc_theme_538())
```

<!--html_preserve-->

<script type="application/json" data-for="htmlwidget-55c45560b14e0547ca08">{"x":{"hc_opts":{"title":{"text":"Top 10 countries by incarceration rate"},"yAxis":{"title":{"text":"Rate per 100,000 population"}},"credits":{"enabled":false},"exporting":{"enabled":true},"plotOptions":{"series":{"turboThreshold":0},"treemap":{"layoutAlgorithm":"squarified"},"bubble":{"minSize":5,"maxSize":25}},"annotationsOptions":{"enabledButtons":false},"tooltip":{"delayForDisplay":10},"subtitle":{"text":"Source: International Centre for Prison Studies"},"xAxis":{"categories":["Seychelles","U.S.A.","St Kitts & Nevis","Turkmenistan","Virgin Is. (USA)","Cuba","El Salvador","Guam (USA)","Thailand","Belize"]},"series":[{"data":[799,698,607,583,541,510,492,469,461,449],"name":"Incarceration rate","type":"bar","color":"#f16913"}],"legend":{"enabled":false}},"theme":{"colors":["#FF2700","#008FD5","#77AB43","#636464","#C4C4C4"],"chart":{"backgroundColor":"#F0F0F0","plotBorderColor":"#606063","style":{"fontFamily":"Roboto","color":"#3C3C3C"}},"title":{"align":"left","style":{"fontWeight":"bold"}},"subtitle":{"align":"left"},"xAxis":{"gridLineWidth":1,"gridLineColor":"#D7D7D8","labels":{"style":{"fontFamily":"Unica One, sans-serif","color":"#3C3C3C"}},"lineColor":"#D7D7D8","minorGridLineColor":"#505053","tickColor":"#D7D7D8","tickWidth":1,"title":{"style":{"color":"#A0A0A3"}}},"yAxis":{"gridLineColor":"#D7D7D8","labels":{"style":{"fontFamily":"Unica One, sans-serif","color":"#3C3C3C"}},"lineColor":"#D7D7D8","minorGridLineColor":"#505053","tickColor":"#D7D7D8","tickWidth":1,"title":{"style":{"color":"#A0A0A3"}}},"tooltip":{"backgroundColor":"rgba(0, 0, 0, 0.85)","style":{"color":"#F0F0F0"}},"legend":{"itemStyle":{"color":"#3C3C3C"},"itemHiddenStyle":{"color":"#606063"}},"credits":{"style":{"color":"#666"}},"labels":{"style":{"color":"#D7D7D8"}},"legendBackgroundColor":"rgba(0, 0, 0, 0.5)","background2":"#505053","dataLabelsColor":"#B0B0B3","textColor":"#C0C0C0","contrastTextColor":"#F0F0F3","maskColor":"rgba(255,255,255,0.3)"},"conf_opts":{"global":{"Date":null,"VMLRadialGradientURL":"http =//code.highcharts.com/list(version)/gfx/vml-radial-gradient.png","canvasToolsURL":"http =//code.highcharts.com/list(version)/modules/canvas-tools.js","getTimezoneOffset":null,"timezoneOffset":0,"useUTC":true},"lang":{"contextButtonTitle":"Chart context menu","decimalPoint":".","downloadJPEG":"Download JPEG image","downloadPDF":"Download PDF document","downloadPNG":"Download PNG image","downloadSVG":"Download SVG vector image","drillUpText":"Back to {series.name}","invalidDate":null,"loading":"Loading...","months":["January","February","March","April","May","June","July","August","September","October","November","December"],"noData":"No data to display","numericSymbols":["k","M","G","T","P","E"],"printChart":"Print chart","resetZoom":"Reset zoom","resetZoomTitle":"Reset zoom level 1:1","shortMonths":["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"],"thousandsSep":" ","weekdays":["Sunday","Monday","Tuesday","Wednesday","Thursday","Friday","Saturday"]}},"type":"chart","fonts":["Roboto","Unica+One"],"debug":false},"evals":[],"jsHooks":[]}</script>
<!--/html_preserve-->
<br>

The final plot also uses the highcharter package to create an interactive map.

``` r
library(highcharter) ; library(RColorBrewer)

data(worldgeojson)

n <- 4
dclass <- data_frame(to = 0:n/n, color = brewer.pal(5, "PuRd"))
dclass <- list.parse2(dclass)
```

``` r
highchart() %>% 
  hc_title(text = "Incarceration rates") %>% 
  hc_subtitle(text = "Source:  International Centre for Prison Studies") %>%
  hc_add_series_map(worldgeojson, df, name = "Rate per 100,000 pop.", 
                    value = "rate", joinBy = "iso3") %>% 
  hc_colorAxis(stops = dclass) %>% 
  hc_legend(enabled = TRUE) %>% 
  hc_mapNavigation(enabled = TRUE)
```
<br>

### Communicating data

There are number of ways to share the results of your data analysis. For example, the [Shiny](http://shiny.rstudio.com/) package allows you to create interactive visualisations in a web browser. There are examples of great Shiny apps in RStudio's [showcase of users' apps](https://www.rstudio.com/products/shiny/shiny-user-showcase/).

[R Markdown](http://rmarkdown.rstudio.com/) is another package that helps you to author dynamic documents, presentations, and reports within R. There are a range of possible output formats including MS Word, PDF, and HTML.

To publish one of your visualisation online on [rpubs.com](http://rpubs.com/) just install and load the [knitr](https://cran.r-project.org/web/packages/knitr/index.html) package, create a new R Markdown document, insert the code into a chunk, click *Knit to HTML*, and press the *Publish* button in the preview window.

### Useful references

-   [RStudio's cheatsheet for dplyr and tidyr](http://www.rstudio.com/wp-content/uploads/2015/02/data-wrangling-cheatsheet.pdf)

-   Garrett Grolemund and Hadley Wickham's [*R for Data Science*](http://r4ds.had.co.nz/)

-   [Tufte in R](http://motioninsocial.com/tufte/)

<br> <br>

<a name="myfootnote1">1</a>: The messiness is a result of my scraping the data from a PDF file.

<br> <br> <br> <br> <br> <br>

##### Answers

1.  How many people are held in penal institutions worldwide?

``` r
summarise(df, total = sum(prison_pop))
```

    ## # A tibble: 1 × 1
    ##      total
    ##      <int>
    ## 1 10307084

1.  What is the prison population of the U.S.A?

``` r
filter(df, country == "U.S.A.") %>% select(prison_pop)
```

    ## # A tibble: 1 × 1
    ##   prison_pop
    ##        <int>
    ## 1    2217000

1.  Which country has the highest prison population rate?

``` r
arrange(df, desc(rate)) %>% slice(1)
```

    ## # A tibble: 1 × 6
    ##      country  iso3         region national_pop prison_pop  rate
    ##        <chr> <chr>         <fctr>        <int>      <int> <dbl>
    ## 1 Seychelles   SYC Eastern Africa        92000        735   799

1.  Which country has the second highest prison population rate?

``` r
arrange(df, desc(rate)) %>% slice(2)
```

    ## # A tibble: 1 × 6
    ##   country  iso3        region national_pop prison_pop  rate
    ##     <chr> <chr>        <fctr>        <int>      <int> <dbl>
    ## 1  U.S.A.   USA North America    317760000    2217000   698

1.  Which country has the lowest prison population rate?

``` r
arrange(df, rate) %>% slice(1)
```

    ## # A tibble: 1 × 6
    ##         country  iso3         region national_pop prison_pop  rate
    ##           <chr> <chr>         <fctr>        <int>      <int> <dbl>
    ## 1 Guinea Bissau   GNB Western Africa      1725000         92     5

1.  What is the world prison population rate?

``` r
summarise(df, total_rate = sum(prison_pop) / sum(as.numeric(df$national_pop)) * 100000) %>% 
  round(0)
```

    ## # A tibble: 1 × 1
    ##   total_rate
    ##        <dbl>
    ## 1        144

1.  What is the median incarceration rate for Oceania?

``` r
filter(df, region == "Oceania") %>% 
  summarise(median = median(rate))
```

    ## # A tibble: 1 × 1
    ##   median
    ##    <dbl>
    ## 1    155

1.  If the U.S.A. has 4% of the world's population what percent of the world's prison population does it have?

``` r
df %>%
  mutate(total_national_pop = sum(as.numeric(national_pop)),
         total_prison_pop = sum(prison_pop)) %>%
  filter(country == "U.S.A.") %>%
  summarise(usa_national_pop_percent = round(national_pop/total_national_pop*100, 1),
    usa_prison_pop_percent = round(prison_pop/total_prison_pop*100, 1))
```

    ## # A tibble: 1 × 2
    ##   usa_national_pop_percent usa_prison_pop_percent
    ##                      <dbl>                  <dbl>
    ## 1                      4.4                   21.5
