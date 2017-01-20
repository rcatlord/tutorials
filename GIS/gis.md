Using R as a GIS
================

This tutorial will show you how R can be used as a Geographical Information System (GIS). R is able to read geospatial vector data (i.e. points, lines, and polygonss) in ESRI's Shapefile format and packages like rgeos, ggmap and leaflet give it similar functionality to a proprietary GIS.

<br>

**The following common GIS routines will be demonstrated in this tutorial**

-   reading spatial data
-   reprojecting
-   clipping
-   spatial joins
-   point in polygon
-   choropleth mapping
-   writing shapefiles

<br>

**The data used in this tutorial**
An ESRI shapefile of London borough boundaries was downloaded from the [London DataStore](http://data.london.gov.uk/dataset/statistical-gis-boundary-files-london). The shapefile is projected in British National Grid and originally derives from the [Ordnance Survey](https://www.ordnancesurvey.co.uk/business-and-government/products/opendata-products.html).

Point data are supplied from [data.police.uk](http://data.police.uk) and represent incidents of anti-social behaviour and crimes recorded by the Metropolitan and City of London Police during May 2016. The incidents are supplied with latitude and longitude coordinates which have undergone an [anonymisation process](http://data.police.uk/about/#anonymisation).

<br>

### Reading shapefiles

**Read the London borough boundary shapefile**
Each shapefile contains feature geometry (.shp), attributes (.dbf) and projection information (.prj). The `readOGR()` function from the [rgdal](https://cran.r-project.org/web/packages/rgdal/index.html) package loads all of these different layers. The first argument refers to the directory location and the second is the file name without suffix. NB You will need to have already set the working directory where the shapefile is stored using `setwd()`.

``` r
library(rgdal)
boroughs <- readOGR(".", "London_Borough_Excluding_MHW", verbose = FALSE)
```

**Inspect the first 4 rows of the attribute data**

``` r
head(boroughs@data, 4)
```

    ##                   NAME  GSS_CODE  HECTARES NONLD_AREA ONS_INNER
    ## 0 Kingston upon Thames E09000021  3726.117      0.000         F
    ## 1              Croydon E09000008  8649.441      0.000         F
    ## 2              Bromley E09000006 15013.487      0.000         F
    ## 3             Hounslow E09000018  5658.541     60.755         F

The $ is used to extract a particular variable from the attribute table.

``` r
boroughs$NAME
```

    ##  [1] Kingston upon Thames   Croydon                Bromley               
    ##  [4] Hounslow               Ealing                 Havering              
    ##  [7] Hillingdon             Harrow                 Brent                 
    ## [10] Barnet                 Lambeth                Southwark             
    ## [13] Lewisham               Greenwich              Bexley                
    ## [16] Enfield                Waltham Forest         Redbridge             
    ## [19] Sutton                 Richmond upon Thames   Merton                
    ## [22] Wandsworth             Hammersmith and Fulham Kensington and Chelsea
    ## [25] Westminster            Camden                 Tower Hamlets         
    ## [28] Islington              Hackney                Haringey              
    ## [31] Newham                 Barking and Dagenham   City of London        
    ## 33 Levels: Barking and Dagenham Barnet Bexley Brent Bromley ... Westminster

**Plot the borough boundaries**

``` r
plot(boroughs, border = "blue")
```

![](gis_files/figure-markdown_github/unnamed-chunk-4-1.png)

**Load the crime data**

``` r
mps <- read.csv("2016-05-metropolitan-street.csv", header = T)
colp <- read.csv("2016-05-city-of-london-street.csv", header = T)
```

**Combine the Metropolitan and City of London Police data, rename the variables, and remove rows with missing coordinates**

``` r
library(dplyr)
crimes <- bind_rows(mps, colp) %>%
  select(long = Longitude,
         lat = Latitude,
         category = Crime.type) %>% 
  mutate(category = factor(category)) %>% 
  filter(!is.na(long)) %>% 
  as.data.frame()
rm(mps, colp)
```

<br>

### Reprojecting

**Convert crimes to a SpatialPointsDataFrame with latlong projection**

``` r
library(sp)
coords <- SpatialPoints(crimes[,c("long","lat")])
crimes <- SpatialPointsDataFrame(coords, crimes)
proj4string(crimes) <- CRS("+init=epsg:4326")
```

**Transform the borough boundary to geographic coordinates (latitude/longitude)**

``` r
boroughs <- spTransform(boroughs, CRS("+init=epsg:4326"))
```

**Check the projection**

``` r
boroughs@proj4string
```

    ## CRS arguments:
    ##  +init=epsg:4326 +proj=longlat +datum=WGS84 +no_defs +ellps=WGS84
    ## +towgs84=0,0,0

**Plot the London borough boundaries and the crime data**
A number of the incidents are geocoded well beyond London's borough boundaries

``` r
plot(crimes)
plot(boroughs, border = "blue", add = T)
```

![](gis_files/figure-markdown_github/unnamed-chunk-10-1.png)

<br>

### Clipping

**Clip the crimes to the borough boundaries**

``` r
proj4string(crimes) <- proj4string(boroughs)
crimes <- crimes[boroughs, ]
```

**Plot the London borough boundaries and the crime data**
The points are now located within the Greater London boundary.

``` r
plot(crimes)
plot(boroughs, border = "blue", add = T)
```

![](gis_files/figure-markdown_github/unnamed-chunk-12-1.png)

<br>

### Spatial joins

Join attributes from the borough boundary polygon layer to the crime data points layer.

``` r
borough_attributes <- over(crimes, boroughs[,c("NAME", "GSS_CODE")])
crimes$borough <- borough_attributes$NAME
crimes$census_code <- borough_attributes$GSS_CODE
```

Calculate the frequency of categories of crime by borough

``` r
crimes %>% as.data.frame() %>% 
  count(borough, category) %>%  
  head()
```

    ## Source: local data frame [6 x 3]
    ## Groups: borough [1]
    ## 
    ##                borough                  category     n
    ##                 <fctr>                    <fctr> <int>
    ## 1 Barking and Dagenham     Anti-social behaviour   550
    ## 2 Barking and Dagenham             Bicycle theft    11
    ## 3 Barking and Dagenham                  Burglary   122
    ## 4 Barking and Dagenham Criminal damage and arson   199
    ## 5 Barking and Dagenham                     Drugs    72
    ## 6 Barking and Dagenham               Other crime    18

<br>

### Points in polygon

Subset the Bicycle theft offences and count the number of points within the London boroughs using the `over()` function

``` r
bicycle_theft <- crimes[crimes$category == "Bicycle theft",]
proj4string(bicycle_theft) <- proj4string(boroughs)
pointsinpolygon <- over(SpatialPolygons(boroughs@polygons), SpatialPoints(bicycle_theft), returnList = TRUE)
boroughs$bicycle_theft <- unlist(lapply(pointsinpolygon, length))
```

<br>

### Choropleth mapping

Calculate thematic ranges using natural breaks using the `classIntervals()` function from the [classInt](https://cran.r-project.org/package=classInt) package. Breaks based on equal counts, quantiles, standard deviation can also be chosen.

``` r
library(classInt)
classes <- classIntervals(boroughs$bicycle_theft[boroughs$bicycle_theft != 0], n=5, style="jenks")
```

Fortify the borough shapefile for plotting in ggplot2 / ggmap

``` r
library(ggplot2) ; library(maptools)
boroughs.f <- fortify(boroughs, region="NAME")
boroughs.f <- left_join(boroughs.f, boroughs@data, by = c("id" = "NAME"))
```

Plot a choropleth map of borough-level bicycle theft in [ggplot2](https://cran.r-project.org/web/packages/ggplot2/index.html)

``` r
library(ggmap) ; library(RColorBrewer)
ggplot() +
  geom_polygon(data = boroughs.f, aes(x = long, y = lat, group = group,
        fill = bicycle_theft), color = "white", size = 0.2) +
  coord_map() +
  scale_fill_gradientn('Frequency\n', colours=brewer.pal(5,"RdPu"), breaks = classes$brks) +
  guides(title = "Offences") +
  theme_nothing(legend = TRUE) +
  labs(title = "Police recorded Bicycle theft offences by borough\n(May 2016)")
```

    ## Warning: `panel.margin` is deprecated. Please use `panel.spacing` property
    ## instead

![](gis_files/figure-markdown_github/unnamed-chunk-18-1.png)

<br>

Plot an interactive choropleth map of borough-level Bicycle theft with an OpenStreetMap raster layer using [leaflet](https://rstudio.github.io/leaflet/)

``` r
library(leaflet)
boroughs_popup <- paste0("<strong>Borough: </strong>",
                               boroughs$NAME,
                               "<br><strong>Offences (May 2016): </strong>",
                               boroughs$bicycle_theft)

colcode <- findColours(classes, c("#feebe2", "#fbb4b9", "#f768a1", "#c51b8a", "#7a0177"))

# make labels for legend
breaks <- round(classes$brks, 1)

labels = matrix(1:(length(breaks)-1))
for(j in 1:length(labels )){labels [j] =
  paste(as.character(breaks[j]),"-",as.character(breaks[j+1]))}

leaflet(data = boroughs) %>% 
  addProviderTiles("OpenStreetMap.BlackAndWhite") %>%
  addPolygons(data = boroughs, 
             fillColor = colcode, 
             fillOpacity = 0.6, 
             color = "#636363", 
             weight = 2, 
             popup = boroughs_popup)  %>%
  addLegend(position = "bottomright",
            colors = c("#feebe2", "#fbb4b9", "#f768a1", "#c51b8a", "#7a0177"),
            labels = labels,
            opacity = 0.8,
            title = "Frequency of offences")
```

<br>

### Writing shapefiles

To write the vector layer with updated attribute table as a shapefile execute the following:

``` r
writeOGR(boroughs, ".", "boroughs", driver="ESRI Shapefile")
```

<br>

### Learning more

Bivand, R. S., Pebesma, E. J., & Rubio, V. G. (2013). [Applied spatial data analysis with R](http://www.asdar-book.org). Springer. 2nd ed.

Brunsdon, Chris, and Lex Comber. (2015). [An Introduction to R for Spatial Analysis and Mapping](https://uk.sagepub.com/en-gb/eur/an-introduction-to-r-for-spatial-analysis-and-mapping/book241031). London: Sage.
