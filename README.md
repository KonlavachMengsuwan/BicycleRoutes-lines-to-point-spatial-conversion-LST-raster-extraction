# BicycleRoutes-lines-to-point-spatial-conversion-LST-raster-extraction

## The code in this repository is designed to be both easy to understand and easy to use. While it is functional, there is always room for improvement and potential for streamlining. We hope to continually refine and optimize the codebase as needed.

## Load library
```
library(rgeos)
library(rgdal)
library(raster)
library(sp)
library(tidyverse)
```

## Set Work directory
```
setwd("")
```

## Read raster files
```
## Read DEM
DEM <- raster("DEM/Lusatia_DEM_UTM.tif")

## Read LST
LST = raster("Landsat8-9/2022_07_19_Landsat8_L1/Results/LST_2022_07_19_Landsat8L1_Lusatia_UTM.tif")
```

## Choose the raster!
```
## Check CRS
raster_file = LST
```

## Read SpatialLinesDataFrame data
```
## Lausitz & Brandenburg Tourism websites
Niederlausitzer_Bergbautour = readOGR("Bicycle_Routes/1_Niederlausitzer Bergbautour/1_Niederlausitzer Bergbautour_tracks_geom.shp")
DahmeRadweg = readOGR("Bicycle_Routes/2_DahmeRadweg/2_DahmeRadweg_tracks.shp")
Elsterradtour = readOGR("Bicycle_Routes/4_Elsterradtour/4_Elsterradtour_tracks_geom.shp")
KohleWindWasser = readOGR("Bicycle_Routes/5_Kohle-Wind und Wasser-Radtour/5_Kohle-Wind und Wasser-Radtour_tracks_geom.shp")
FürstPücklerWeg = readOGR("Bicycle_Routes/6_Fürst-Pückler-Weg/6_Fürst-Pückler-Weg_tracks_geom.shp")
SchwarzeElsterRadweg = readOGR("Bicycle_Routes/7_Schwarze-Elster-Radweg/7_Schwarze-Elster-Radweg_tracks_geom.shp")
Gurkenradweg = readOGR("Bicycle_Routes/8_Gurkenradweg/8_Gurkenradweg_tracks_geom.shp")
Hofjagdweg = readOGR("Bicycle_Routes/9_Hofjagdweg/9_Hofjagdweg_tracks_geom.shp")
OderNeißeRadweg = readOGR("Bicycle_Routes/10_Oder-Neiße-Radweg/10_Oder-Neiße-Radweg_tracks_geom.shp")
Spreeradweg = readOGR("Bicycle_Routes/11_Spreeradweg/11_Spreeradweg_tracks_geom.shp")
TourBrandenburg = readOGR("Bicycle_Routes/12_Tour Brandenburg/12_Tour Brandenburg_tracks_geom.shp")
SeenlandRoute = readOGR("Bicycle_Routes/13_Seenland-Route/13_Seenland-Route_tracks_geom.shp")

## Tobias
FuerstPuecklerWeg = readOGR("Bicycle_Routes/Tobias/fuerst-pueckler-weg_tracks_geom.shp")
Mühlenrundtour = readOGR("Bicycle_Routes/Tobias/Mühlenrundtour_tracks_geom.shp")
NiederlausitzerBergbautour = readOGR("Bicycle_Routes/Tobias/Niederlausitzer Bergbautour_tracks_geom.shp")
SpreewälderFünfSchlösser = readOGR("Bicycle_Routes/Tobias/Spreewälder Fünf-Schlösser-Radtour_tracks_geom.shp")
Tour1 = readOGR("Bicycle_Routes/Tobias/Tour1_tracks_geom.shp")
```

## Put all Bicycle Routes into a list
```
Bicycle_Line = list()

Bicycle_Line[[1]] = Niederlausitzer_Bergbautour
Bicycle_Line[[2]] = DahmeRadweg
Bicycle_Line[[3]] = Elsterradtour
Bicycle_Line[[4]] = KohleWindWasser
Bicycle_Line[[5]] = FürstPücklerWeg
Bicycle_Line[[6]] = SchwarzeElsterRadweg
Bicycle_Line[[7]] = Gurkenradweg
Bicycle_Line[[8]] = Hofjagdweg
Bicycle_Line[[9]] = OderNeißeRadweg
Bicycle_Line[[10]] = Spreeradweg
Bicycle_Line[[11]] = TourBrandenburg
Bicycle_Line[[12]] = SeenlandRoute
Bicycle_Line[[13]] = FuerstPuecklerWeg
Bicycle_Line[[14]] = Mühlenrundtour
Bicycle_Line[[15]] = NiederlausitzerBergbautour
Bicycle_Line[[16]] = SpreewälderFünfSchlösser
Bicycle_Line[[17]] = Tour1
```

## How many Bicycle Routes?
```
n_line = 17
```

## Transform the coordination system to UTM
```
utm_proj <- CRS("+init=epsg:32633")

for(i in 1:n_line)
{
  Bicycle_Line[[i]] <- sp::spTransform(Bicycle_Line[[i]], utm_proj)
}
```

## Calculate the length of the line
```
route_length = data.frame(matrix(nrow = n_line, ncol = 1))
colnames(route_length) = c("route_length")

for(i in 1:n_line)
{
  route_length[i,1] = rgeos::gLength(Bicycle_Line[[i]])
}
```

## Specify the desire distance of each points
```
distance = 10 # m 

point_distance = list()

for(i in 1:n_line)
{
  point_distance[[i]] = seq(0, route_length[i,1], by = distance)
}
```

## Create points from the line
```
points_matrix = list()

for(i in 1:n_line)
{
  points_matrix[[i]] <- rgeos::gInterpolate(Bicycle_Line[[i]], point_distance[[i]])
}
```

## Convert SpatialPoints to SpatialPointsDataframe
```
for(i in 1:n_line)
{
  points_matrix[[i]]  = as(points_matrix[[i]] ,"SpatialPointsDataFrame")
}
```

## Create data frame table 
```
for(i in 1:n_line)
{
  points_matrix[[i]]@data = data.frame(matrix(nrow = length(point_distance[[i]]), ncol = 6))
  colnames(points_matrix[[i]]@data) = c("LINE_ID", "ID", "DIST", "X", "Y", "Z")
}
```

## Insert information to the data frame table
```
for(i in 1:n_line)
{
  #points_matrix[[i]]@data$LINE_ID = sub("Route: ", "", Bicycle_Line[[i]]$name)
  
  points_matrix[[i]]@data$ID = 1:length(point_distance[[i]])
  points_matrix[[i]]@data$X = points_matrix[[i]]@coords[,1]
  points_matrix[[i]]@data$Y = points_matrix[[i]]@coords[,2]
  points_matrix[[i]]@data$DIST = seq(0, route_length[i,1], by = distance)
  points_matrix[[i]]@data$Z = raster::extract(raster_file, points_matrix[[i]])
}
```

## Plot the extracted value over the distance
```
i = 17
ggplot(points_matrix[[i]]@data, aes(x = DIST, y = Z)) +
  geom_line()
```

## Export the SpatialPointsDataframe as shapefile
```
getwd()

## Export SpatialLinesDataFrame as shapefile

writeOGR(obj=points_matrix[[1]], dsn="Bicycle_Routes/1_Niederlausitzer Bergbautour", layer="1_Niederlausitzer Bergbautour_tracks_points_UTM", driver="ESRI Shapefile") 
writeOGR(obj=points_matrix[[2]], dsn="Bicycle_Routes/2_DahmeRadweg", layer="2_DahmeRadweg_tracks_points_UTM", driver="ESRI Shapefile") 
writeOGR(obj=points_matrix[[3]], dsn="Bicycle_Routes/4_Elsterradtour", layer="4_Elsterradtour_tracks_points_UTM", driver="ESRI Shapefile") 
writeOGR(obj=points_matrix[[4]], dsn="Bicycle_Routes/5_Kohle-Wind und Wasser-Radtour", layer="5_Kohle-Wind und Wasser-Radtour_tracks_points_UTM", driver="ESRI Shapefile") 
writeOGR(obj=points_matrix[[5]], dsn="Bicycle_Routes/6_Fürst-Pückler-Weg", layer="6_Fürst-Pückler-Weg_tracks_points_UTM", driver="ESRI Shapefile") 
writeOGR(obj=points_matrix[[6]], dsn="Bicycle_Routes/7_Schwarze-Elster-Radweg", layer="7_Schwarze-Elster-Radweg_tracks_points_UTM", driver="ESRI Shapefile") 
writeOGR(obj=points_matrix[[7]], dsn="Bicycle_Routes/8_Gurkenradweg", layer="8_Gurkenradweg_tracks_points_UTM", driver="ESRI Shapefile") 

writeOGR(obj=points_matrix[[8]], dsn="Bicycle_Routes/9_Hofjagdweg", layer="9_Hofjagdweg_tracks_points_UTM", driver="ESRI Shapefile") 
writeOGR(obj=points_matrix[[9]], dsn="Bicycle_Routes/10_Oder-Neiße-Radweg", layer="10_Oder-Neiße-Radweg_tracks_points_UTM", driver="ESRI Shapefile") 
writeOGR(obj=points_matrix[[10]], dsn="Bicycle_Routes/11_Spreeradweg", layer="11_Spreeradweg_tracks_points_UTM", driver="ESRI Shapefile") 
writeOGR(obj=points_matrix[[11]], dsn="Bicycle_Routes/12_Tour Brandenburg", layer="12_Tour Brandenburg_tracks_points_UTM", driver="ESRI Shapefile") 
writeOGR(obj=points_matrix[[12]], dsn="Bicycle_Routes/13_Seenland", layer="13_Seenland-Route_tracks_points_UTM", driver="ESRI Shapefile") 

# Tobias
writeOGR(obj=points_matrix[[13]], dsn="Bicycle_Routes/Tobias", layer="fuerst-pueckler-weg_tracks_points_UTM", driver="ESRI Shapefile") 
writeOGR(obj=points_matrix[[14]], dsn="Bicycle_Routes/Tobias", layer="Mühlenrundtour_tracks_points_UTM", driver="ESRI Shapefile") 
writeOGR(obj=points_matrix[[15]], dsn="Bicycle_Routes/Tobias", layer="Niederlausitzer Bergbautour_tracks_points_UTM", driver="ESRI Shapefile") 
writeOGR(obj=points_matrix[[16]], dsn="Bicycle_Routes/Tobias", layer="Spreewälder Fünf-Schlösser-Radtour_points_UTM", driver="ESRI Shapefile") 
writeOGR(obj=points_matrix[[17]], dsn="Bicycle_Routes/Tobias", layer="Tour1_tracks_points_UTM", driver="ESRI Shapefile") 
```
