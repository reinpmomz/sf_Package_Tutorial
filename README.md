# sf_Package_Tutorial
This is a tutorial covering the sf R package

install.packages(c("tidyverse", "data.table", "fpp3","GGally","sugrrants", 
                 "prophet", "rpart", "rpart.plot", "forecast", "gt"))

---
title: "sf: A Tutorial"
output: 
    learnr::tutorial:
      css: css/styles.css
runtime: shiny_prerendered
---


```{r setup, include=FALSE}
library(learnr)
knitr::opts_chunk$set(echo = TRUE,
                      comment = "",
                      warning = FALSE)
```

## Introduction

This is a work-in-progress tutorial covering the `sf` R package. `sf` provides a powerful interface for R users to work with geospatial data stored in vector formats. 

The name comes from *simple features* or *simple feature access* which refers to a formal standard (ISO 19125-1:2004) that describes how objects in the real world can be represented in computers, with emphasis on the spatial geometry of these objects. (This description taken verbatim from the first package vignette.)

  


#### Data 

This tutorial uses several data files which are saved to a [github repository](https://github.com/thisisdaryn/data/tree/master/geo). In some instances, small changes have been made e.g. a change of file format, removing attributes from a data set. Each of these files was originally acquired from the [Chicago Data Portal](https://data.cityofchicago.org/) 

| File Name      | Description           | Original Data Source  |
|---------------|:-------------|------:|
|Chi_Community_Areas.geojson | Community area boundaries | https://data.cityofchicago.org/Facilities-Geographic-Boundaries/Boundaries-Community-Areas-current-/cauq-8yn6  |
|CTA_Bus_Routes.geojson |CTA Bus Routes| https://data.cityofchicago.org/Transportation/CTA-Bus-Routes-Shapefile/d5bx-dr8z|
| CTA_Bus_Stops.geojson |CTA Bus Stops| https://data.cityofchicago.org/Transportation/CTA-Bus-Stops/hvnx-qtky |
|Public_Health_Indicators.csv||https://data.cityofchicago.org/Health-Human-Services/Public-Health-Statistics-Selected-public-health-in/iqnk-2tcu|


#### Resources

  - [Geocomputation with R](https://geocompr.robinlovelace.net/) - Robin Lovelace, Jakub Nowosad, Jannes Muenchow 
  
  - sf: package vignettes 
    1. [Simple Features for R](https://r-spatial.github.io/sf/articles/sf1.html)
    2. [Reading, Writing and Converting Simple Features](https://r-spatial.github.io/sf/articles/sf2.html)
    3. [Manipulating Simple Feature Geometries](https://r-spatial.github.io/sf/articles/sf3.html)
    4. [Manipulating Simple Features](https://r-spatial.github.io/sf/articles/sf4.html)
    5. [Plotting Simple Features](https://r-spatial.github.io/sf/articles/sf5.html)
    6. [Miscellaneous](https://r-spatial.github.io/sf/articles/sf6.html)




## Reading in data 


First read in the `sf::st_read`. 

### Chicago Community Areas

```{r input1, message = FALSE}
library(sf)
comms <- st_read("https://raw.githubusercontent.com/thisisdaryn/R_tutorials/master/data/Chicago_Community_Areas.geojson") 
```

#### Contemplating the information shown on screen

Upon reading the data, several pieces of information are output and are worth contemplating/understanding. In particular:

  * <i>geometry type</i>, what type of geometric entities are represented in this data. In this case, the value <i>MULTIPOLYGON</i> tells us what types of geographic entities are represented in the data.
  
  * <i>bbox</i>, a bounding box that contains the geographic elements described in the data 

  * <i>CRS</i>, [Coordinate Reference System (CRS)](. The value, <i>4326</i> indicates that this data uses the [WGS84](http://epsg.io/4326) CRS. CRSs determine how assigned values map to locations on the earth.
  
  

  
  
### Chicago Transit Authority Bus Routes
  

Next, we read in data describing the different bus routes offered by the Chicago Transit Authority (CTA):

```{r}
bus_routes <- st_read("https://raw.githubusercontent.com/thisisdaryn/R_tutorials/master/data/CTA_Bus_Routes.geojson")
```

Take note of some key differences between this output and the previous including:

  1. *geometry type*: MULTILINESTRING (as opposed to MULTIPOLYGON in the case of the community boundaries)
  
  2. *bbox*: these values look much different from those of the previous data frame. In fact they do not look like conventional longitude and latitude values at all. 

  2. *CRS*: 3435. This indicates that this data uses the [NAD83/Illinois East](http://epsg.io/3435) reference system. 

This illustrates that geospatial data can be encoded using multiple reference systems. 

**In order to work with data sets using different CRSs you should take care to convert all your data to a shared CRS.** 

One should not assume that coordinate values will represent longitude and latitude. (Also, even if coordinates are encoded in longitude and latitude values one should pay attention to the CRS. If you are working with multiple data sets it is important to be aware of which reference systems they are each coming from. Overlooking this may lead to incorrect computations.)



```{r}
bus_routes
```


We can see that there are several columns:

  * *ROUTE*: the route number
  
  * *NAME* the route name
  
  * *WKDAY*, whether or not (1/0) the bus runs on weekdays
  
  * *SAT*, whether or not the bus runs on Saturdays
  
  * *SUN*, whether or not the bus runs on Sundays
  
  * *SHAPE_LEN*, the length of the route 
  
  * *geometry* column holding the spatial descriptions of the bus routes


Similar to before, we can access the coordinates of a single geometry entry using `[[`. 

```{r showgeomroute, exercise = TRUE}
bus_routes$geometry[[1]]
```


### Chicago Libraries


Last, we will read in a data file with the locations of vehicles that were found abandoned in Chicago in 2016. 

```{r}
libraries <- st_read("https://raw.githubusercontent.com/thisisdaryn/R_tutorials/master/data/Chicago_Libraries.geojson")
```
Just as before 
  
  * the geometry type is POINT, indicating that each each location is a single point
  
```{r}
print(libraries)
```


  
## A closer look at sf objects  


First, use `class` to find out what class of object, <i>comms</i> is. 

```{r}
class(comms)
```


We see that <i>comms</i> is of class `sf` but also `data.frame`. This means that it can be used as a data frame would in other computations.


An `sf` object is essentially a data frame with a special geometry column that contains geospatial data. Other columns contain values that are termed *attributes*. 

First, let's look at *comms*

```{r}
comms
```

If you scroll all the way over to the end there is a column named *geometry* and each element in this column is of type `sfc_MULTIPOLYGON`. That means that each items is a set of polygons. 


#### Inspecting a MULTIPOLYGON closely

Depending on where you have run the command you may be able to get a glimpse of what the data in the *geometry* column looks like. If you want to get a closer look at what the data in the *geometry* column you can explicitly select it from by running the code below. The output may seem like a lot but you will probably be able to identify values that are longitude and latitude pairs in Chicago, Illinois.

```{r showgeomelement, exercise = TRUE}
comms$geometry[[1]]
```


## Getting the CRS and bounding box of an sf object

The `sf::st_crs` and `sf::st_bbox` functions return the CRS and bounding box of an sf object respectively.   

```{r}
st_crs(comms)
st_bbox(comms)
```



## sf built-in plotting 


`sf` objects have a default plotting function that can be invoked by using the `plot` function with the object as input. For example, we can plot the data in *comms*:

```{r}
plot(comms)
```

`plot` creates a set of maps: one for each attribute in the data that is not the geometry. The interpretability or usefulness of these plots will vary with the nature of the attributes. 


**Exercise**: Use `plot` to generate plots of *bus_routes* and *vehicles_pts*. How should the resulting plots be interpreted? 

```{r sfplotex, exercise = TRUE}

```
## Transforming data from one CRS to another 

Before we move on, remember that we have data utilising different CRSs:

  * Data using WGS84 CRS
    * the map of community areas 
    * abandoned vehicle locations
    
  * data using NAD83 CRS 
    * map of CTA bus routes

We will transform the data in the <i>bus_routes</i> data frame from NAD83 to WGS84 before moving forward.  

  * `sf::st_transform` will be used to transform the data in *bus_routes* to an equivalent representation in another CRS. `st_transform` requires two inputs: an sf object and a crs specification.

  * `sf::st_crs` will be used to get the CRS of *comms* to be used as the necessary 2nd input to `sf::st_transform`

```{r}
wgs_routes <- st_transform(bus_routes, st_crs(comms))
print(wgs_routes)
```


You can compare the coordinates in this new data frame to the previous if you wish. 

```{r showgeomwgsroutes, exercise = TRUE}
wgs_routes$geometry[[1]]
```



##  Plotting using ggplot2 


Next, we create our first plot with the data, using the `ggplot2` package. 

If you are familiar with `ggplot2` the code below will look very similar to code for plots you have made before. e use the *comms* data frame as the data input and `ggplot2::geom_sf` as the geometry.  


```{r}
library(ggplot2)

ggplot(data = comms) + 
  geom_sf()
```



**Exercise**: Make a plot showing the CTA's Bus Routes using code similar to 

```{r plotroutes, exercise = TRUE}

```

```{r plotroutes-solution}
ggplot(data = bus_routes) +
  geom_sf()

```


**Exercise**: Make a plot showing the locations of all libraries in Chicago


```{r plotlibraries, exercise = TRUE}

```

```{r plotlibraries-solution}
ggplot(data = libraries) +
  geom_sf()
```









## Subsetting data 

`sf` objects are data frames and can be subsetted like data frames. For instance if we wanted to restrict the data to the communities of Rogers Park, West Ridge, Uptown, Lincoln Square and Edgewater, we could do so using `dplyr::filter` (or with base R subsetting). 


```{r message = FALSE}
library(dplyr)

community_group <- c("ROGERS PARK", "WEST RIDGE", "UPTOWN", "LINCOLN SQUARE", "EDGEWATER")
small_map <- filter(comms, community %in% community_group)
print(small_map)

#alternatively we could have used the following base R syntax
#small_map <- comms[comms$community %in% community_group, ]
```


**Exercise**: Plotting this restricted set of communities can be done similarly to before (The code below introduces a new additional geometry):

```{r}
ggplot(data = small_map) + 
  geom_sf() +
  geom_sf_text(aes(label = community))
```


**Exercise**: In the box below, write code to make a map showing only the <i>111A</i> bus route:

```{r plot111A, exercise = TRUE}
route111a <- filter(routes_wgs, ROUTE == "111A")

ggplot(data = route111a) + 
  geom_sf()
```

```{r plot111A-solution, exercise = TRUE}
route111A <- filter(wgs_routes, ROUTE == "111A")

ggplot() + 
  geom_sf(data = route111A) 
```





## Binary Predicates 


Next we will look at some of the binary predicate functions offered by the `sf` package. These functions allow us to compute if specified relationships exist between geospatial entities. These functions all start with the `st_` and have full names reflecting the relationship they are intended to identify. 

The list of binary predicate functions available in `sf` are: 


  * `st_intersects`

  * `st_disjoint`

  * `st_touches`

  * `st_crosses`

  * `st_within`

  * `st_contains`

  * `st_contains_properly`

  * `st_overlaps`

  * `st_equals`

  * `st_covers`

  * `st_covered_by`

  * `st_equals_exact`

  * `st_is_within_distance`


We will use some of these in examples in the upcoming sections.

### Example: st_intersects with x and y inputs

The most commonly-used of the binary predicate functions may be `st_intersects`. The function name implies that it can be used to determine if geospatial entities intersect.  

First, let's try `st_intersects` with two inputs. 

**Question: For each community area, can we identify the bus routes that intersect with it?**

Here we can provide *comms* and *wgs_routes* as arguments to `st_intersects`:

```{r}
st_intersects(comms, wgs_routes)
```

**Understanding this output** 

The output above can be interpreted as:

  * The community in the first row of the *comms* object intersects with the routes in rows 5, 18, 29, 71, 78, 81, 87, 96, 108 and 113 of the *wgs_routes* object


  * The community in the second row of the *comms* object intersects with the routes in rows 3, 13, 53, 71, 81, 88, 96, and 101 of the *wgs_routes* object
  
  
  * etc etc etc 
  
  
**Note that the numbers in the output denote the row numbers in the input data object. Not the assigned number of the bus routes.** 


### Example: st_intersects with x input only


If the *y* argument is not specified, `st_intersects` - and all the other binary predicate functions -use *x* as the second argument as well. This has the effect of finding which features in *x* intersect with each other.

**Question: Which bus routes intersect with each other?**


```{r}
route_intersections <- st_intersects(wgs_routes)
route_intersections
```

**Note that each bus route is listed as intersecting with itself.**


### Example: st_touches

You may also want to know which communities border a given community. Here we can use `st_touches`. 

**Question: which community areas border the Albany Park community?**

```{r}
albany_park <- filter(comms, community == "ALBANY PARK")
albany_neighbours <- st_touches(albany_park, comms)
albany_neighbours
```

We can get the names of these neighbouring areas:
```{r}
comms$community[albany_neighbours[[1]]]
```



All of this may raise a question: could `st_intersects` have been used in this situation as well? 

```{r}
st_intersects(albany_park, comms)
```

Here the output is very similar but there is one more community in the list of neighbors. This is the Albany Park community itself.








**Question: Which libraries does each community area cover?**

Here we can use `st_covers`:

```{r}
st_covers(comms, libraries)
```

Alternatively, we could have used `st_covered_by` with the order of the arguments reversed:

```{r}
st_covered_by(libraries, comms)
```

This reflects the same information but presents the information


### A note on other binary predicate functions

The previous examples show only some of the binary predicate functions available in `sf`. Each of the binary predicate functions not discussed use similar syntax and are named in way that their use cases are self-hinting - if not self-evident. 


## Brief detour: A note about left joins and inner joins

Before we move on to spatial joins, we will review briefly the concepts of left joins and inner joins. These may be unfamiliar to you. Here we will take time to give a small example using a data set that is not geospatial (though it does contain geographic information).

For this example we will read in two data sets with the populations and areas of some countries respectively:

```{r message = FALSE}
library(readr)
areas <- read_csv("https://raw.githubusercontent.com/thisisdaryn/data/master/Areas.csv")
populations <- read_csv("https://raw.githubusercontent.com/thisisdaryn/data/master/Populations.csv")
```

```{r}
areas
```

We can see that both of these tables have a field named *Country*. Some countries are listed in both tables while others are only listed in one of the tables.

First, we will do a left join using `dplyr::left_join`:

```{r}
left_join(populations, areas)
```

We can see that the left join keeps all the countries in the *populations* table in the final output even if there's no available area information for those countries. Countries that are only in the *areas* table do not show up at all in the final table.

Now, let's try an inner join using `dplyr::inner_join`:

```{r}
inner_join(populations, areas)
```

Here we see that only the countries that are in both tables show up in the final table. 

The spatial joins we do using `sf::st_join` will be analogous to either the left joins or the inner joins we have done here.

**Note**: When joining data, it is not always going to be the case that the joins will be as simple as the above example. For common : the keys may have different names in different data sets, the keys may be stored as different data types in different data sets, or muliple columns/variables may be required to be used as keys.

## Spatial Joins 

Often when working with geospatial data, we need to bring data from 2 (or more) data sets together based on a some articulation of shared geographic location. An instance of this process is known as a **spatial join**. 

Spatial joins are driven can be carried out by `sf::st_join` and are driven by the binary predicate functions we have just been introduced to. 

`sf::st_join` takes multiple arguments. We will use 4 key arguments:
  
  1. *x*: an sf object
  
  2. *y*: another sf object (defaults to *x* if this is not satisfied)
  
  3. *join*: a binary predicate function that indicates the spatial relationship to base the join on. (defaults to `st_intersects`)
  
  4. *LEFT*: indicates whether or not (TRUE/FALSE) a left join or an inner join is intended. (defaults to TRUE i.e. a left join is performed)
  




### Example: Adding community area information to the libraries

How can we find all the vehicles that were abandoned? Let's use `st_join` with the following inputs:

  - *x*: *libraries*
  
  - *y*: *comms*
  


```{r}
libraries_comms <- st_join(libraries, comms)
```


We can look at the joined object:

```{r echo = FALSE}
libraries_comms
```

  - The joined object has columns from both of the input objects

  - has a single geometry column, of type POINT, from the first input
  
  - there is one row for each row row in *libraries*
  
  - some communities show up multiple times 
  


```{r}
comms_libraries <- st_join(comms, libraries)
comms_libraries
```

  - this joined data frame has columns from both *libraries* and *comms*
  
  - the geometry column is taken from *comms*
  
  - some rows have `NA` values for all the fields that were originally in `comms`
  
  - some communities are in multiple rows of the joined data frame even though they were in only one row of `comms`
  
#### Key observation: `st_join` retains the geometry of the first input. 

If you want to keep rows in this table that do not have matching rows in the second input you must do a left join (the default, *left* = `TRUE`). If you wish to keep only the rows that are matched with rows in the second input you must do an inner join i.e. set *left* = `FALSE`. 


#### Exercise: Which communities have no libraries? 

```{r nolibs, exercise = TRUE}
no_libraries <- st_join(comms, libraries) %>% 
  filter(is.na(name_))

ggplot(data = no_libraries) + 
  geom_sf()


```


#### Exercise:  Which communities does route 2 pass through?**
  
**Make a map of all the communities and route *2***
  
```{r show54b, exercise = TRUE}

```



```{r show54b-solution}
route2 <- filter(wgs_routes, ROUTE == "2")
comms2 <- st_join(comms, route2, st_intersects, left = FALSE)

ggplot() + 
  geom_sf(data = comms2) +
  geom_sf(data = route2, color = "blue") + 
  geom_sf_text(data = comms2, aes(label = community))
```

## Spatial aggregation
