<img width="91" alt="GAlogo" src="https://user-images.githubusercontent.com/101831333/179605423-3d161bbb-bdaf-4d23-8cd1-f931262f74cb.png">

# London real estate rent market
## - Impact of different parameters for the prediction of the rent prices -

### Overview
This project was completed as part of the General Assembly Data Science Immersive bootcamp. This document discusses the problem, hypothesis, methodology, conclusion, and tools used.

Repository Contents:
0. Readme
1. Jupyter notebook for scraping of real estate data
2. Jupyter notebook for additional features
3. Jupyter notebook for EDA
4. Jupyter notebook for modelling

Please note the data files are not included

### Problem Statement
Predicting the rent level for real estate objects is one of the key factors which everyone would prefer to do before he is renting a new flat or investing funds into real estate. Due to my background within the real estate sector I decided to analyse the real estate rental market in London. Nowadays with several real estate websites it is possible to filter out the best results for your needs. The websites already have a lot of information about the real estate objects, but I wanted to see if additional features would help to predict the rent level even better.

### Data collection
#### Basic real estate data
For the first information of the real estate objects, I decided to scrape the data from the website zoopla.co.uk. To avoid getting captures I created a little function to change the user agent every time when I send a new request. Afterwards I used beautiful soup combined with RegEx searches to extract the following information from Zoopla for each object.
- Title of announcement
- Rent price
- Location
- Publishing date
- Amount of bedrooms
- Amount of bathrooms
- Amount of receptions
- Seize
- Description
- Longitude
- Latitude
- Close by public transport
- Link to the single overview of the object

My scraper was running the first time on 10.05.2022 and collected approximately 47.000 results. The scraper can be found in the jupyter notebook “scraping of real estate data”. Afterwards I will run the scraper on a monthly basis and update the results.

### Additional features
Besides the above listed features from Zoopla, I would like to include additional features to see if they would increase the scores of my models. The collection of the below information can be found in the jupyter notebook “additional features”

##### Gyms
As a first additional feature I would like to include if a gym is close by to the real estate object. To collect the information about the gyms I used the API from Yelp to gather several information about the gyms in London. My focus was lying on the location of the gyms. But the API provided even more information for example the price categories.

##### Supermarkets
Also, for the supermarkets I would like to check if and which market is close to the real estate object. Again, I used the yelp API to collect the location of the following supermarkets within London:
Tesco
Marks and Spencer
Sainsbury’s
Whole Foods

##### Public Transport
The information collected from Zoopla already included information about public transportation nevertheless I decided to check on my own which public transportations is close by. Like this I can set my own radius to decide which stations are classified as close by. I found the website https://www.doogal.co.uk/london_stations.php which already collected all London tube and train stations with their coordinates.

### Data Cleaning
The additional features didn’t need any cleaning for now. The longitude and latitude were included and together with the name this was the main information that I needed.
The real estate data on the other side needed some additional cleaning:
Prices need to reflect a monthly rent
Interpreting continuous and categorical features. Adjust the type if needed
Creating more model-friendly feature names.
Exploring opportunities for new feature creation.
Extracting postcodes from location column

### EDA
#### Missing values
Some of the features included no information. I adjusted this and either dropped the values or included 0.

<img width="380" alt="missing values" src="https://user-images.githubusercontent.com/101831333/179606173-9ee8f9fa-33f7-4b10-80f3-e6a2f47a60fe.png">

#### Outliers
Especially three sections especially showed some outliers: the price, the longitude and the latitude.

<img width="454" alt="outliers" src="https://user-images.githubusercontent.com/101831333/179606393-f12f7340-83f8-4041-b13d-801f3b59392b.png">
For the price I was not surprised especially within some locations in London you can find very expensive apartments or houses. I decided for the price to filter the results for a range of plus and minus 3 standard deviations around the median. My intention was to receive more representative results.

For the longitude and latitude, it turned out that Zoopla collected some locations outside of London. I excluded this to only include objects within the greater area of London.

### Correlations
The correlation heatmap showed the correlation for all numerical features. The number of bedrooms is showing the highest correlation with the price. Surprisingly the longitude and the latitude showed no big impact.

<img width="292" alt="correlation" src="https://user-images.githubusercontent.com/101831333/179606544-d9640fdd-7575-4337-9c49-f15c9ed0f40b.png">

Especially the low impact of the longitude and latitude made me curious. Normally the location should show the highest impact. I decided to calculate the mean price per postcode:

<img width="200" alt="correlation" src="https://user-images.githubusercontent.com/101831333/179606883-e04ce1d8-4b00-41b9-830f-41311ba2bcb7.jpg">

It clearly shows that the location has an impact on the rent level. I will investigate this further within the modelling process.

### Feature engineering
After reviewing and cleaning the basic data I would like to include my additional features. For this I created a function which is using the longitude and latitude of the real estate objects and the additional features to calculate the following:
- Per supermarket how many of them are within a range of 1 km
- How many gyms are within a range of 1 km
- How many train and tube stations are within a range of 1 km
- which tube and train stations are within a range of 1 km

This information is added to each object of my initial data from Zoopla. Afterwards I checked again the correlation of the features with the price per month:

<img width="325" alt="corr_list" src="https://user-images.githubusercontent.com/101831333/179607193-78be011f-152a-459c-8320-6ed9bbe89f8d.png">

My additional features are included but I will need to analyse this more in details during the modelling process.

### Modelling
The target variable (the rent price) is a continuous variable. Because of this I will fit regression models. In a first instance I will run four different once:
- Basic Linear Regression (without penalties)
- A regression with Lasso penalty
- A regression with Ridge penalty
- An ElesticNet which covers a combination of both penalties (Lasso and Ridge)

The penalties of the lasso and ridge model will reduce the variance, but this will also raise the errors/bias. I will use the inbuilt cross validation functions to find the right bias / variance trade-off for the best compromise.
The two different penalties have a different approach. I would like to mention that with the Lasso penalty it is more likely that some of the coefficients will be zero at the end.

I will run the models with different parameters/features to identify the impact.

#### Basic Zoopla data with Postcodes or Coordinates
Within the EDA process I realised that the longitude and latitude are not showing a big correlation whereas the postcode does. Within a first modelling process I the following information which are available on Zoopla:
- Amount of Bedrooms
- Amount of Bathrooms
- Amount of Receptions
- Close by Public transport stations

Afterwards I either included the coordinates or the postcode to compare how the scores of the models will be affected:

<img width="454" alt="result_1" src="https://user-images.githubusercontent.com/101831333/179607549-18170e8c-5a96-43fc-8838-ce72aa05a036.png">

For the valuation I will look at the cross-validation score. We can clearly see that the postcodes are bringing much better results compared to the coordinates. This surprised me as the coordinates are much more precise than the postcodes. But the longitude and latitude and two different features. I assume only a combination of longitude and latitude will bring a better result. I would like to have a closer look at this after my course at GA.

I had a closer look at the best scored model RidgeCV with postcodes and the biggest impact of price was coming from the number of bedrooms. Below an overview of the prediction’s vs the true values form the test set:

<img width="454" alt="residuals" src="https://user-images.githubusercontent.com/101831333/179607656-89c1a2d8-7e44-408e-920b-fbe835b14440.png">

#### Modelling with additional features
Within a next stepped I used the data with the postcodes and added my additional features to run the same models again:

<img width="454" alt="results 2" src="https://user-images.githubusercontent.com/101831333/179607806-b79eee43-ff21-4cf4-9755-466ee38d6302.png">

The complete date showed the highest score, but the impact is not very high. From the correlation results within the EDA I expected actually a higher impact. I reviewed the coefficients of the features of the ridge model with the complete data.

<img width="221" alt="add_features" src="https://user-images.githubusercontent.com/101831333/179608016-8f479a97-cd83-45fa-9748-68e3de89fafd.png">

None of them showed a big impact but was surprisingly for me is that the number of whole foods close by are showing a negative impact. I didn’t expect this as, for me, this is a supermarket for which I expect it would be located in more posh areas.

#### Additional models
After the regression models I decided to rune some - additional models to analyse the features:
- Stats model for analysis of coefficients
- KNeighborsRegressor
- DecisionTreeRegressor
- RandomForestRegressor

The random forest regressor scored actually the best and within all models and here my additional features showed also an impact:


<img width="454" alt="feature_importance" src="https://user-images.githubusercontent.com/101831333/179607827-71a866d8-edda-4d47-9866-b04931a8ec99.png">

## Conclusion
For normal regression models the additional features showed no big impact but within the random forest regressor the impact was very high. We need to keep in mind that the models are working differently. The regression models try to fit a best line whereas the random forest regressors tries to find the best allocation based on the features. I assume as more gyms and supermarkets are located within the centre of London for this reason it’s a good criteria for the regressor to evaluate which objects are closer to the centre.

## Limitations
I assume the yelp API didn’t provide me with the location of all supermarkets and gyms within London.

## Key learning
Throughout this project I learned how to use my learned skills on a real-world problem. From collecting over cleaning to analysing the data. The more I was working with the data the more I thought what else I can do with it (more under future work section).

## Future work
During the preparation of the project I realised that I would like to have a closer look at the data after my course and even increase the amount observations and features:
- I would like to collect the data on a monthly basis to analyse the rent over a longer timeframe to identify upcoming areas
- Include additional features like crime rate or more location for supermarkets
- Work again with longitude and latitude
- Using additional models, for example TensorFlow.

## Libraries used
- Pandas
- NumPy
- Scikit-learn
- Matplotlib
- Seaborn
- Geopandas
- Geopy
- scipy
