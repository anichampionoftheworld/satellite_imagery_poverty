
# Capstone - Predicting Poverty From Satellite Imagery
<img 
src="https://github.com/anichampionoftheworld/satellite_imagery_poverty/blob/main/assets/satellite.png" width="300" height="200">

---

### Contents:
- [Problem Statement](#Problem-Statement)
- [Background](#Background)
- [Datasets](#Datasets)
- [Data Visualization and Analysis](#Data-Visualization-&-Analysis)
- [Conclusions and Recommendations](#Conclusions-&-Recommendations)
- [Sources](#Sources)

---

### Problem Statement: 

Can satellite imagery be used to predict poverty in US cities? While fairly granular data on poverty in the US already exists, thanks to the US census, it is only reported once every 10 years and cannot produce statistically reliable wealth estimates for areas smaller than several square blocks. For this reason, high-resolution satellite imagery - in particular, the Sentinel satellite, which transmits images in which each pixel represents just 10 square meters of the Earth's surface - could be very helpful in assessing more nuanced and precise delineations of poverty. Additionally, it can be difficult for the US census to find and ask questions of the most impoverished members of our community, because they often don't have permanent addresses at which they can send and recieve mail or census personnel. A well-calibrated model built off satellite data could serve as a useful tool to capture people how they are, as opposed to trying to track them down using more conventional methods that don't match up with how people actually live. Finally, poverty does not look the same everywhere. This project will use one US city's imagery to predict poverty in other city. The accuracy of these predictions will be able to tell us if there are any similarities between different centers of urban poverty, and if continued study would be useful.

To solve this problem we will look at Chicago and DC satellite data, and try to predict poverty within each city. We will also test the DC-trained model on the Chicago data and vice versa.

---

### Background:

In the last ten years, economists have begun using satellite images of Earth at night - commonly called "night lights" - to assess economic activity in different countries. Each pixel in these images stores a number that represents the relative brightness of lights reflected back from Earth. Economists have published many successful papers predicting national wealth over time by using the increase in pixel brightness as a proxy for growing electricty usage and therefore GDP. The key issue with these meausurements, however, is that the images are fairly low resolution - each pixel covers a square kilometer of land, and sub-national assessments have not proven particularly effective. 

Different types of satellite imagery have been used by remote sensing analysts to make agricultural assessments for many years. Satellites that capture certain bands of light can be used to show soil aridity, moisture in the air, types of vegetation growth, and other key indiciators. The type of satellite that this project uses is called Sentinel 2, which was launched in June 2015 by the European Space Agency. It sends back images with information on 12 spectral bands (as opposed to the more limited 3 bands, usually red, green, and blue), with each pixel representing just 10 square meters of land.

Machine learnists have successfully built models on similar types of data to perform image segmentation to identify buildings, illegal trash dumps, destruction from hurricanes, and many other uses. These projects often involve many thousands of images to train models, and require massive computing power because of the size of raster imagery. This project will be much more scoped in that it will only use a single (composited) image for each city, using a training set of pixels to predict the rest of the pixels in the image. 

---

### Datasets:

This project contains three different types of data: income data in a normal pandas dataframe, satellite imagery stored as a raster file, and shapefiles of each census block group for both DC and Chicago.

**Income Data:** 
The US has fairly comprehensive and laborious data-collection methods for studying its own population - the Census, collected every 10 years, primary among them. Additionally, the American Community Survey (ACS) collects data on a more frequent basis, and publishes all of this data in a publically acessible data portal with an API. For this project I used the Censusdata python wrapper to pull just the median household income data for each census block group (smaller than a census tract but bigger than an individual census block, census block groups are the smallest area for which the ACS reports income). 

**Census Block Group Polygons:**
Both Chicago and DC have data portals from which I was able to pull shapefiles for each of the census block groups in each city. Shapefiles are somewhat complex geospatial files that are actually five different files that all most be downloaded together. The geopandas library can be used to read in just the .shp file, as long as the other files are also present, and display it in a normal-looking dataframe. The critial piece of information here is the "geometry" column, which stores the latitue/longitude coordinates that form the shape of each observed area segment. Seperately, I also had to use a Chicago boundary shapefile to make sure that I was only looking at Chicago census and satellite data, and not data for all of Cook County.

**Satellite Imagery:**
The most complicated imagery acquisition was the satellite images of each city. Initially, I attempted to use the Sentinel2 API to capture an image of DC by submitting DC's geometry as a parameter, but I could not get an image that covered the entirety of DC's area. Therefore I turned to Google Earth Engine (https://code.earthengine.google.com/), which is mostly written in Java, to extract better imagery. This tool has the benefit of enabling one to build a composite image of a specific area, and also to mask cloud coverage. I created a composite image each of DC and Chicago by average the value for each pixel across 6 months of imagery data (which comes to about 35 images). These images were stored tif files, and each stored a pixel value across 12 spectral bands. Each pixel is also associated with a lat/long coordinate to locate it on the earth's surface.

*The two satellite images as well as the predictions on Chicago data are two large for github to handle, and therefore will not be uploaded to this repo.*

---

### Data Visualization & Analysis:

Much of the initial data visualization and analysis in this project concerned the building and extraction of satellite imagery. 
- The first stage involved acquiring the census shapefiles, and projecting them on to a common open source map using the folio python library: 
<img src="https://github.com/anichampionoftheworld/satellite_imagery_poverty/blob/main/assets/DC_folio.png" width="600" height="400">
- Once I connected the income data to the shapefiles, I was able to plot the median household income data along with the geometry of the census block groups. Here you can see that the north-western areas of DC are substantially wealthier, and that the southeast quadrant contains some of the poorest areas in the city by far. The average median household income in DC almost hits six figures at $99,994.
<img src="https://github.com/anichampionoftheworld/satellite_imagery_poverty/blob/main/assets/income_dc.png" width="500" height="400">
- As a slight aside, for those less familiar with DC, the contrasts in this map looks remarkably similar to the maps portraying residents who had received early doses of the vaccine and new COVID cases (as of February 1st). People living in wealthier areas in DC were decidedly more likely to receive the vaccine, and much less likely to get sick during the worst of the pandemic this winter.
<img src="https://github.com/anichampionoftheworld/satellite_imagery_poverty/blob/main/assets/DC_vaccine.png" width="600" height="300">
<img src="https://github.com/anichampionoftheworld/satellite_imagery_poverty/blob/main/assets/dc_covid.png" width="300" height="200">
- I was also able to plot the income data with the relevent shapefiles for Chicago. The goegraphic shape of poverty is a little different here, with wealth concentrated in the center of the city and povery radiating outward, with higher concentrations of poverty on the south and west sides. The average median household income for Chicago is significantly lower than in DC, at $72,749.
<img src="https://github.com/anichampionoftheworld/satellite_imagery_poverty/blob/main/assets/income_chi.png" width="500" height="500">
- The distribution of income in DC versus the distribution of income in Chicago is quite different - Chicago looks fairly normal with a long right tail, whereas DC has very lumpy data - there are some extremely wealthy census block groups as well as a good chunk under the $50k mark. 
<img src="https://github.com/anichampionoftheworld/satellite_imagery_poverty/blob/main/assets/dist_income_dc.png" width="600" height="400">
<img src="https://github.com/anichampionoftheworld/satellite_imagery_poverty/blob/main/assets/dist_income_chi.png" width="600" height="400">
- The goal of this project was to build a binary classifier that identified regions at or below poverty. Initially I just looked at census block groups where the median income was below the federal poverty line. However, this didn't give me much data to work with, and wasn't really representative of poverty in cities where the cost of living is significantly higher. As a result, I ended up calculating multipling the federal poverty level by the increased cost of living in each city over the national average - 23 percent in Chicago's case and 39 percent in DC's. I then "rasterized" these census tracts to form labels for my model, with each pixel labelled a 1 or a 0. 
<img src="https://github.com/anichampionoftheworld/satellite_imagery_poverty/blob/main/assets/dc_sat_and_income.png" width="400" height="400">
<img src="https://github.com/anichampionoftheworld/satellite_imagery_poverty/blob/main/assets/chi_sat_and_income.png" width="400" height="400">

---

### Conclusions & Recommendations:

Using a neural network, I was able to build a model that was very slightly better than the baseline for DC. Only about 5.7 percent of DC census block groups are below my somewhat improvised poverty level, giving me a baseline score of 94.3 percent. My model predicted with an accuracy of 95.3 percent. Admittedly this is a very small improvement, but looking at it another way it's actually 1/5 closer to perfect! Additionally, it was able to correctly identify 31 percent of the poverty pixels (a recall score of 0.31). A second model built off of oversampled poverty data, trying to correct for these imbalanced classes, actually did quite a bit worse than the original model.

When this model was used to predict on Chicago satellite imagery, it did better in some areas than others. It correctly predicted the densest poverty on the far south side, but entirely missed the poverty on the west side. It also overpredicted on the north side, which only has a few impoverished census block groups. 
<img src="https://github.com/anichampionoftheworld/satellite_imagery_poverty/blob/main/assets/predictions_on_chi.png" width="500" height="500">

Additionally, I also tested two similar models (one with an imbalanced classes correction and one without) going the reverse direction - predicting DC poverty using Chicago data. Both of these models did quite a bit worse than the DC-trained model, which was surprising because Chicago actually had more data to train on. I suspect that the model was thrown off because one of the impoverished DC census block groups actually contains a large chunk of the river, and that affected the analysis. Below are the predictions from this Chicago:
<img src="https://github.com/anichampionoftheworld/satellite_imagery_poverty/blob/main/assets/predictions_on_dc.png" width="500" height="500">

Overall, this project was really more of a test case to see if this workflow was feasible. While it did not produce an incredibly accurate model, it was still able to learn some patterns of poverty that could be generalized across cities, although leaving substantial room for improvement. With more computing power one could run a convolutional neural net off of thousands of images of each city, or one could add more identifying features to each pixel - such as whether it represented a road, a building, or a park. 

---

### Sources:

https://towardsdatascience.com/using-satellites-to-map-economic-opportunity-in-new-york-city-3059aece5404

https://towardsdatascience.com/neural-network-for-satellite-data-classification-using-tensorflow-in-python-a13bcf38f3e1

https://geohackweek.github.io/raster/04-workingwithrasters/

https://medium.com/land-use-land-cover-classification-using-google/land-use-land-cover-classification-using-google-earth-engine-random-forest-algorithm-landsat-and-28b610f555de

https://towardsdatascience.com/land-cover-classification-of-satellite-imagery-using-convolutional-neural-networks-91b5bb7fe808

https://towardsdatascience.com/satellite-imagery-access-and-analysis-in-python-jupyter-notebooks-387971ece84b

https://code.earthengine.google.com/

https://data.cityofchicago.org/

https://opendata.dc.gov/