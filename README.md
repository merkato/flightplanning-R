# flightplanning-R

![license](https://img.shields.io/badge/license-MIT-green.svg) 

An R package for generating UAV flight plans, especially for Litchi.

<img src="man/images/MANEJO_4.0_alta_velocidade.gif" alt="Animation of drone taking photos along the flight plan" align="center"/>

## Installation

This version of package should be installed using the devtools/remotes:

```r
# install.packages("devtools")
devtools::install_github("gsapijaszko/flightplanning-R")
```

## This fork

* adopts the package to R >= 4.2.0 (backward compatible) - DONE
* added `grid = FALSE | TRUE` parameter, which sets the flying direction over polygon - DONE
* added input for `sf` polygons -- `roi` can be read with `sf::st_read()` - DONE
* try to replace {rgdal}, {rgeos} and {sp} with {sf} - DONE; use `litchi_sf()` function
* incorporates changes from Hivemapper (see: https://github.com/caiohamamura/flightplanning-R/pull/4)
* if you prefer to run in in QGIS, please check the [QGIS processing script](https://github.com/gsapijaszko/qgis_r_processing/blob/main/uav_planner_litchi.rsx) available on [qgis r processing repo](https://github.com/gsapijaszko/qgis_r_processing).

## New function for testing
 * `litchi_sf()`

### Example usage

``` r
library(flightplanning)
f <- "mytest/lasek.gpkg"
roi <- sf::st_read(f)

output <- "mytest/fly.csv"

params <- flight.parameters(
  height = 120,
  focal.length35 = 24,
  flight.speed.kmh = 24,
  side.overlap = 0.8,
  front.overlap = 0.8
)

litchi_sf(roi,
  output,
  params,
  gimbal.pitch.angle = -90,
  flight.lines.angle = -1,
  max.waypoints.distance = 400,
  max.flight.time = 18,
  grid = FALSE
)
#> #####################
#> ## Flight settings ##
#> #####################
#> Min shutter speed: 1/128
#> Photo interval:    4 s
#> Flight speed:      23.364 km/h
#> Flight lines angle: 104.3223
#> Total flight time: 16.2097
```

![](https://i.imgur.com/mG1npRr.png)

Per default `litchi_sf()` takes first polygon from the input layer. If there is a need to run a mission over several polygons, you can union them like:

```r
if(nrow(roi) > 1) {
  roi <- sf::st_union(roi)
}

litchi_sf(roi,
          output,
          params,
          gimbal.pitch.angle = -90,
          flight.lines.angle = -1,
          max.waypoints.distance = 400,
          max.flight.time = 18,
          grid = FALSE
)
#> Your flight was splitted in 2 splits,
#> because the total time would be 26.91 minutes.
#> They were saved as:
#> mytest/fly1.csv
#> mytest/fly2.csv
#> The entire flight plan was saved as:
#> mytest/fly_entire.csv
#> #####################
#> ## Flight settings ## 
#> #####################
#> Min shutter speed: 1/128
#> Photo interval:    4 s
#> Flight speed:      23.364 km/h
#> Flight lines angle: 104.2442
#> Total flight time: 26.9055
```
![](https://i.imgur.com/Vfr1Bj9.png)

Using `grid = TRUE` parameter you can change the direction of the grid like:

```r
litchi_sf(roi,
          output,
          params,
          gimbal.pitch.angle = -90,
          flight.lines.angle = -1,
          max.waypoints.distance = 400,
          max.flight.time = 18,
          grid = TRUE
)
```
![](https://i.imgur.com/MRFkkrO.png)

## Usage
There are two main functions available:
 * `flight.parameters()`: this will calculate the flight parameters given desired settings for GSD/height, target overlap, flight speed and camera specifications.
 * `litchi.plan()`: it depends on the `flight.parameters()` return object to generate the CSV flight plan ready to import into the Litchi Hub.
 
### flight.parameters
 - `height`: target flight height, default NA
 - `gsd`: target ground resolution in centimeters, must provide either `gsd` or `height`
 - `focal.length35`: numeric. Camera focal length 35mm equivalent, default 20
 - `image.width.px`: numeric. Image width in pixels, default 4000
 - `image.height.px`: numeric. Image height in pixels, default 3000
 - `side.overlap`: desired width overlap between photos, default 0.8
 - `front.overlap`: desired height overlap between photos, default 0.8
 - `flight.speed.kmh`: flight speed in km/h, default 54.
 
 ### litchi.plan
  - `roi`: range of interest loaded as an OGR layer, must be in
a metric units projection for working properly
 - `output`: output path for the csv file
 - `flight.params`: Flight Parameters. parameters calculated from flight.parameters()
 - `gimbal.pitch.angle`: gimbal angle for taking photos, default -90 (overriden at flight time)
 - `flight.lines.angle`: angle for the flight lines, default -1 (auto set based on larger direction)
 - `max.waypoints.distance`: maximum distance between waypoints in meters,
default 2000 (some issues have been reported with distances > 2 Km)
 - `max.flight.time`: maximum flight time. If mission is greater than the estimated time, 
 it will be splitted into smaller missions.
 - `starting.point`: numeric (1, 2, 3 or 4). Change position from which to start the flight, default 1
 - `grid`: boolean (FALSE | TRUE). Change the fly direction over polygon from parallel to perpendicular
 
## Authors
 - Caio Hamamura
 - Danilo Roberti Alves de Almeida
 - Daniel de Almeida Papa
 - Hudson Franklin Pessoa Veras
 - Evandro Orfanó Figueiredo

This package was developed by author and its contributors which helped providing the calculations and testing.

## Example
``` R
# Install and load the package
install.packages("flightplanning")
library(flightplanning)

params = flight.parameters(height=100,
                          flight.speed.kmh=54,
                          side.overlap = 0.8,
                          front.overlap = 0.8)
                          
params

## Slot "flight.line.distance": 
## [1] 34.61329 
##  
## Slot "flight.speed.kmh": 
## [1] 46.72794 
## 
## Slot "front.overlap":
## [1] 0.8
## 
## Slot "gsd":
## [1] 4.326662
## 
## Slot "height":
## [1] 100
## 
## Slot "ground.height":
## [1] 129.7998
## 
## Slot "minimum.shutter.speed":
## [1] "1/289"
## 
## Slot "photo.interval":
## [1] 2

# Load example SpatialDataFrame polygon
exampleBoundary = readOGR(
                          system.file("extdata", 
                                      "exampleBoundary.shp", 
                                      package="flightplanning"), 
                          "exampleBoundary")

# Set the output
output = "output.csv"

# Create the csv plan 
litchi.plan(exampleBoundary,
            output,
            params,
            flight.lines.angle = -1,
            max.waypoints.distance = 2000,
            max.flight.time = 15)
            
# Your flight was splitted in 2 splits,
# because the total time would be 27.62 minutes.
# They were saved as:
# output1.csv
# output2.csv
# The entire flight plan was saved as:
# output_entire.csv
# #####################
# ## Flight settings ## 
# #####################
# Min shutter speed: 1/289
# Photo interval:    2 s
# Flight speed:      46.7279 km/h
# Flight lines angle: 39.0626
# Total flight time: 27.6239
```
<img src="https://github.com/caiohamamura/flightplanning-R/blob/master/man/images/plot_flightplan.png" alt="Flight plan plot" align="center"/>

## References
FIGUEIREDO, E. O. et al. Planos de Voo Semiautônomos para Fotogrametria com Aeronaves Remotamente Pilotadas de Classe 3. Acre: EMBRAPA. 2018
