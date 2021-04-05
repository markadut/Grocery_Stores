# Point Pattern Analysis of the Grocery Stores in California

## 1. Project Overview
In this project, we will be using point pattern analysis (PPA) to determine the grocery stores in California are distributed equitably. Point pattern analysis, in non-technical terms, is the study of spatial arrangements of points in space. This form of analysis helps us understand whether a certain set of spatial data points are distributed normally, in a clustered fashion, randomly, or regularly. For our analysis, we will be using 3 data sets: the grocery stores data set, its corresponding geographical data set (showing polygon coordinates), and the California Population Density data set. The data sets can be found in the following github page: https://github.com/markadut/Data-Science-Fellow-Brown-University.

## 2. Preparing the Data
There are several different R packages that can be utilized while executing a PPA. In this project, we will be using sf, maptools, rgdal, raster, and spatstat. These packages help us work clean, analyze, and visualize geographical data. 

#load all necessary libraries for point pattern analysis (PPA)

library(sf)
library(maptools)
library(rgdal)
library(raster)
library(spatstat)

setwd("/Users/markadut/Desktop/DATA 1150 FELLOW/3_9_NewFiles") # Replace the file path with the actual location of your shapefiles.

# Load the California Shapefile and convert to type owin
s  <- readOGR(".","CA_boundary")
ca <- as.owin(s)

# Load a groceries.shp point feature shapefile
c  <- readOGR(".", "Groceries")  
groceries <- as(c, "ppp")  # Create .ppp object
marks(groceries)  <- NULL # ppp objects have marks as attributes, but in this project, we won't be using them. Therefore, we remove the marks. 
Window(groceries) <- ca 

# Load a population density raster layer for California
img <- raster("/Users/markadut/Desktop/cali_projected_raster.tif")
pop  <- as.im(img)


In the chunk of code given above, "Window(groceries) <- ca" represents the binding of the 
California shapefile polygon to the groceries point feature object. 
We can visualize this by running the following code: 

![image](https://user-images.githubusercontent.com/77459250/113620065-7cd18780-9662-11eb-93fe-b53d45882189.png)






