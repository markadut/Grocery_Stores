# Point Pattern Analysis of the Grocery Stores in California

## 1. Project Overview
In this project, we will be using point pattern analysis (PPA) to determine the grocery stores in California are distributed equitably. Point pattern analysis, in non-technical terms, is the study of spatial arrangements of points in space. This form of analysis helps us understand whether a certain set of spatial data points are distributed normally, in a clustered fashion, randomly, or regularly. For our analysis, we will be using 3 data sets: the grocery stores data set, its corresponding geographical data set (showing polygon coordinates), and the California Population Density data set. The data sets can be found in the following github page: https://github.com/markadut/Data-Science-Fellow-Brown-University.


---
title: "Point Pattern Analysis (PPA) of Grocery Stores in California"
author: "Mark A. Adut"
date: "3/27/2021"
output:
  html_document: default
  pdf_document: default
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## 1. Project Overview
In this project, we will be using point pattern analysis (PPA) to determine the grocery stores in California are distributed equitably. Point pattern analysis, in non-technical terms, is the study of spatial arrangements of points in space. This form of analysis helps us understand whether a certain set of spatial data points are distributed normally, in a clustered fashion, randomly, or regularly. For our analysis, we will be using 3 data sets: the grocery stores data set, its corresponding geographical data set (showing polygon coordinates), and the California Population Density data set. The data sets can be found in the following github page: https://github.com/markadut/Data-Science-Fellow-Brown-University.

## 2. Preparing the Data
There are several different R packages that can be utilized while executing a PPA. In this project, we will be using sf, maptools, rgdal, raster, and spatstat. These packages help us work clean, analyze, and visualize geographical data. 

```{r groceries, warning=FALSE, message=FALSE}

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

```

In the chunk of code given above, "Window(groceries) <- ca" represents the binding of the 
California shapefile polygon to the groceries point feature object. 
We can visualize this by running the following code: 

```{r grocery store viz}
plot(groceries, main=NULL, cols=rgb(0,0,0,.2), pch=20)
```

<p align="center">
  <img src="images/grocery_stores.png">
</p>

Now, we will play around with the population density raster file. Population density values for an administrative layer are usually quite skewed. The following code chunk generates a histogram from the California population raster layer.

```{r pop raster hist}
hist(pop, main=NULL, las=1) # pop density raster layer histogram
```

Transforming the skewed distribution in the population density covariate helps reveal relationships between point distributions and the covariate in some of the point pattern analyses covered later in this tutorial. Therefore, we create a log-transformed version of the pop variable. 

```{r log of pop raster layer hist}
pop.lg <- log(pop) 
hist(pop.lg, main=NULL, las=1)
```

## 3. Density Based Analysis

## 3.1 Quadrat Density Analysis:

We can compute the quadrat count and intensity using Spatstat’s quadratcount() and intensity() functions. The following code chunk divides the state of California into a grid of 6 rows and 6 columns then tallies the number of points falling in each quadrat. 

```{r quadratcount}
Q <- quadratcount(groceries, nx= 6, ny=6)  # dividing the the CA region into a 6 x 6 grid
plot(groceries, main=NULL, cols="grey70",pch=20) # plotting points
plot(Q, add = TRUE)  # adding quadrat grid
```

Then, we can compute and visualize the density of points within each quadrat as follows:

```{r density/intensity}
Q.d <- intensity(Q)
plot(intensity(Q, image=TRUE), main=NULL, las=1)  # Plot density raster
plot(groceries, pch=20, cex=0.6, col=rgb(0,0,0,.5), add=TRUE)  # Add points
```

The density values are reported as the number of points (stores) per square meters, per quadrat. These small length units are not practical at this scale of an analysis. So, we will rescale our data to km. 

```{r density}
groceries.km <- rescale(groceries, 1000, "km")
ca.km <- rescale(ca, 1000, "km")
pop.km  <- rescale(pop, 1000, "km")
pop.lg.km <- rescale(pop.lg, 1000, "km")
```

Now, we can compute the density for each quadrant (in counts per km2) and visualize the density raster/points. 

``` {r km conversion density plot}
# calculating density for each quadrant
Q  <- quadratcount(groceries.km, nx= 6, ny=6)
Q.d <- intensity(Q)

# Plot the density
plot(intensity(Q, image=TRUE), main=NULL, las=1)  # Plot density raster
plot(groceries.km, pch=20, cex=0.6, col=rgb(0,0,0,.5), add=TRUE)  # Add points
```

## 3.2 Quadrat density on a tessellated surface

We can now execute a quadrat density analysis on tessellated surface. In order to do this, we will first divide the population density covariate into four tessellated surfaces. Tessellation allows us to convert a continuous field into discretized areas. 

In this example, we will not work with the log transformed population density values because this would result in certain blank, disconnected areas due to the negative pixel values associated with the log transform (check log.pop histogram above). 

```{r tessellated}
# we divide the population density covariate into tessellated surfaces
brk  <- c(-Inf, 0, 2, 4, Inf)  # Defining the breaks
Zcut <- cut(pop, breaks=brk, labels=1:4)  # Classifying the raster
E <- tess(image=Zcut)  # Creating the tessellated surface

plot(E, main="", las=1)
```

Next, we will plot the density values across each tessellated region.

```{r}
plot(intensity(Q, image=TRUE), las=1, main=NULL)
plot(groceries.km, pch=20, cex=0.6, col=rgb(1,1,1,.5), add=TRUE) # overlaying grocery stores
```

Modifying the color scheme: 

```{r color scheme }
cl <-  interp.colours(c("lightyellow", "orange" ,"red"), E$n)
plot( intensity(Q, image=TRUE), las=1, col=cl, main=NULL)
plot(groceries.km, pch=20, cex=0.6, col=rgb(0,0,0,.5), add=TRUE)
```
## 3.3 Kernel density raster

The spatstat package has a function called density which computes an isotropic kernel intensity estimate of the point pattern. Its bandwidth defines the kernel’s window extent. The following code chunk uses the default bandwidth.

```{r kernel density raster}
K1 <- density(groceries.km) # Using the default bandwidth
plot(K1, main=NULL, las=1)
contour(K1, add=TRUE)
```

In this next chunk, a 150 km bandwidth (sigma = 150) is used. Note that the length unit is extracted from the point layer’s mapping units (which was rescaled to kilometers earlier in this tutorial).

```{r sigma50}
K2 <- density(groceries.km, sigma=150) # Using a 150km bandwidth
plot(K2, main=NULL, las=1)
contour(K2, add=TRUE)
```

The kernel defaults to a gaussian smoothing function. The smoothing function can be changed to a quartic, disc or epanechnikov function. For example, to change the kernel to a disc function type:

```{r epanechnikov}
K3 <- density(groceries.km, kernel = "disc", sigma=150) # Using a 150km bandwidth
plot(K3, main=NULL, las=1)
contour(K3, add=TRUE)
```

## 3.5 Modeling intensity as a function of a covariate

The relationship between the predicted grocery store point pattern intensity and the population density distribution can be modeled following a Poisson point process model. We’ll generate the Poisson point process model then plot the results.

```{r modeling intensity, warning=FALSE}
# Create the Poisson point process model
PPM1 <- ppm(groceries.km ~ pop.km)
# Plot the relationship
a <- plot(effectfun(PPM1, "pop.km", se.fit=TRUE), main=NULL, 
     las=1)

```
Note that this is not the same relationship as ρ vs. population density shown in the previous section. Here, we are fitting a well defined model to the data whose parameters can be extracted from the PPM1 object.

```{r PPM1}
#returning the Nonstationary Poisson Process Analysis Results
PPM1
```

Here, the model takes form:
                                  λ(i) = e^(−13.71 + 1.27)

The base intensity is close to zero (e−13.71) when the logged population density is zero and for every increase in one unit of the logged population density, the grocery point density increases by e^1.27 units.

## 5. Distance based analysis

## 5.1 Average nearest neighbor analysis: 

In this portion of the tutorial, we will be executing an average nearest neighbor (ANN) analysis. ANN is a tool that helps measure the distance between each feature centroid location. Later, it averages all the nearest neighbor distances. If the calculated mean is less than the average for a hypothetical random distribution, the distribution of the features are considered clustered. (ESRI)

```{r ANN}
# setting the 1st nearest neighbor distance to 1
mean(nndist(groceries.km, k=1))
```

```{r ANN2}
# setting the 1st nearest neighbor distance to 1
mean(nndist(groceries.km, k=2))
```

The parameter k can take on any order neighbor (up to n-1 where n is the total number of points). 

The average nearest neighbor function can be expended to generate an ANN vs neighbor order plot. In the following example, we’ll plot ANN as a function of neighbor order for the first 100 closest neighbors:

```{r}
ANN <- apply(nndist(groceries.km, k=1:100),2,FUN=mean)
plot(ANN ~ eval(1:100), type="b", main=NULL, las=1, xlab = "neighbor order number",  ylab = "ANN (km)")
```


As could be seen from the plot above, as the number of nearest neighbors are increased, the ANN (km) value also increases. This is an increasing function with a rate similar to that of a y = sqrt(x) function. 

## 5.2 K and L functions: 

```{r Kestfunction}
K <- Kest(groceries.km)
plot(K, main=NULL, las=1)
```

The plot returns different estimates of K depending on the edge correction chosen. By default, the isotropic, translate and border corrections are implemented. To learn more about these edge correction methods type ?Kest at the command line. The estimated Kfunctions are listed with a hat ^. The black line (Kpois) represents the theoretical K function under the null hypothesis that the points are completely randomly distributed (CSR/IRP). Where K falls under the theoretical Kpois line the points are deemed more dispersed than expected at distance r. Where K falls above the theoretical Kpois line the points are deemed more clustered than expected at distance r.

Now, we can plot the L function as well as the Lexpected function

```{r Lestfunction}
L <- Lest(groceries.km, main=NULL)
plot(L, main=NULL, las=1)
```

plotting the L function with the Lexpected line set horizontally: 

```{r LvsLexpected}
plot(L, . -r ~ r, main=NULL, las=1)
```

## 5.3 Analysis using Pair Correlation Function (pcf)
 
```{r pcf}
g  <- pcf(groceries.km)
plot(g, main=NULL, las=1)
```

As with the Kest and Lest functions, the pcf function outputs different estimates of g using different edge correction methods (Ripley and Translate). The theoretical g-function gPois under a CSR process (green dashed line) is also displayed for comparison. Where the observed g is greater than we can expect more clustering than expected and where the observed g is less than gPois we can expect more dispersion than expected. 


## 6. Hypothesis tests

#  6.1 Testing for clustering/dispersion


```{r mean.ndist}
ann.p <- mean(nndist(groceries.km, k=1))
ann.p
```
The observed average nearest neighbor distance is 4.778 km. 

setting the null model for our simulation:
```{r montecarlo}
n <- 599L               # setting the number of simulations
ann.r <- vector(length = n) # creating empty object to store simulation results (ANN values)
for (i in 1:n){
  rand.p   <- rpoint(n=groceries.km$n, win=ca.km)  # generating random point locations
  ann.r[i] <- mean(nndist(rand.p, k=1))  # Tally the ANN values
}
```

In the above loop, the function rpoint is passed two parameters: n=groceries.km and win=ca.km. The first tells the function how many points to randomly generate (groceries.km$n extracts the number of points from object groceries.km). The second tells the function to confine the points to the extent defined by ca.km. Note that the latter parameter is not necessary if the ca boundary was already defined as the groceries window extent.

We can visualize a randomly placed grocery stores California plot by running the following chunk of code: 

```{r }
plot(rand.p, pch=16, main=NULL, cols=rgb(0,0,0,0.5))
```

Our observed distribution of grocery stores certainly does not look like the outcome of a completely independent random process. As could be seen from the map above, the distribution of grocery stores don't appear to be independently random. It is evident that there are certain clusterings across the region. 

Next, we'll’s plot the histogram of expected values under the null and add a blue vertical line showing where our observed ANN value lies relative to this distribution.

```{r }
hist(ann.r, main=NULL, las=1, breaks=40, col="bisque", xlim=range(ann.p, ann.r))
abline(v=ann.p, col="blue")
```

Now, we will be running a ANN analysis for the 2nd order process underlying a point pattern thus requiring that we control for the first order (pop density dist). Here, we will pass the parameter f=pop.km instead of f=ca.km. By changing the parameter to this, we will be telling R to recognize the pop.km raster to be used to define where a point should most likely be placed. 

```{r}
n <- 599L
ann.r <- vector(length=n)
for (i in 1:n){
  rand.p  <- rpoint(n=groceries.km$n, f=pop.km)  # substituting ca.km with pop.km
  ann.r[i] <- mean(nndist(rand.p, k=1))
}
```

Now, you can plot the last realization of the non-homogeneous point process to convince yourself that the simulation correctly incorporated the covariate raster in its random point function.

```{r}
Window(rand.p) <- ca.km  # Replace raster mask with ma.km window
plot(rand.p, pch=16, main=NULL, cols=rgb(0,0,0,0.5))
```

Next, let’s plot the histogram and add a blue line showing where our observed ANN value lies.

```{r}
hist(ann.r, main=NULL, las=1, breaks=40, col="bisque", xlim=range(ann.p, ann.r))
abline(v=ann.p, col="blue")
```

Even though the distribution of ANN values we would expect when controlled for the population density nudges closer to our observed ANN value, we still cannot say that the clustering of grocery stores can be explained by a completely random process when controlled for population density.

## 6.2 Computing a pseudo p-value from the simulation

A (pseudo) p-value can be extracted from a Monte Carlo simulation. We will work off of the last simulation. First, we need to find the number of simulated ANN values greater than our observed ANN value.

To compute the p-value, find the end of the distribution closest to the observed ANN value, then divide that count by the total count. Note that this is a so-called one-sided P-value. See lecture notes for more information.


```{r hypotest results}
N.greater <- sum(ann.r > ann.p)
p <- min(N.greater + 1, n + 1 - N.greater) / (n +1)
p
```
In our working example, it is evident that the simulated ANN value was nowhere near the range of ANN values computed under the null yet we don’t have a p-value of zero, indicating that the strength of our estimated p value be proportional to the number of simulations. 

## 6.3 Test for a poisson point process model with a covariate effect

The ANN analysis addresses the 2nd order effect of a point process. Here, we’ll address the 1st order process using the poisson point process model.

We’ll first fit a model that assumes that the point process’ intensity is a function of the logged population density (this will be our alternate hypothesis).

```{r testing for poisson point process, warning=FALSE}

PPM1 <- ppm(groceries.km  ~ pop.km) # note that if we use pop.lg
PPM1

```

Next, we’ll fit the model that assumes that the process’ intensity is not a function of population density (the null hypothesis).

```{r null_hypo}
PPM0 <- ppm(groceries.km ~ 1) 
PPM0
```

In this testing for poisson point process model with a covariate effect, we are using this null (homogenous intensity)

model:            
                                     λ(i) = e^-4.795
                                      
λ(i) under the null is nothing more than the observed density of grocery stores within the State of California, or:
  
```{r dividing by area}
groceries.km$n / area(ca.km)  
```

The alternate model will take the form:
                                    
λ(i) = e^(−13.71 + 1.27) - note that the power is the logged pop density
 
We can then compare the models using the likelihood ratio test (LRT):

```{r LRT, message=FALSE, warning=FALSE}
anova(PPM0, PPM1, test="LRT")
```

In this table, the value under the heading PR(>Chi) gives us the probability that we would be wrong in rejecting the null. Given that for our data this value reads 2.2e-16, a number very close to zero, the model suggests that there is close to a 0% chance that we would be wrong in rejecting the base model in favor of the alternate model. This means that, the pop density data does a better job in explaining the distribution of grocery stores in the region. 


## References:

Baddeley, Adrian, Ege Rubak, and Rolf Turner. 2016. Spatial Point Patterns, Methodology and Applications with R. Florida: CRC Press.

Manuel Gimond, https://mgimond.github.io/Spatial/point-pattern-analysis-in-r.html
                            
United States Census - TIGER line Shapefiles 2018. 













