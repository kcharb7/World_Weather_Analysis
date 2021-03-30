#WeatherPy with Python APIs
## Overview
### *Purpose*
PLANMYTRIP is a travel technology company that specializes in internet services for the hotel and lodging industry. Jack, the Head of Analysis for the User Interface Team, asked me to help provide real-time suggestions for clients’ ideal hotels based on a given range of latitude and longitude, as well as the right kind of weather. This was to be completed by collecting and analyzing weather data across 500 cities worldwide. 

## Analysis
### *Create Latitude and Longitude Combinations*
To start, I created a new Jupyter Notebook file called “WeatherPy.ipynb”. Within my new file, I imported the Pandas, Matplotlib, and NumPy dependencies:
```
# Import the dependencies.
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
```

Chaining the NumPy module to the random module, I created code to generate 1,500 random latitudes and longitudes as floating-point decimal numbers. I started with 1,500 to ensure coordinates were fairly distributed around the world. Then I used the zip() function to pack the latitudes and longitudes as pairs:
```
# Create a set of random latitude and longitude combinations.
lats = np.random.uniform(low=-90.000, high=90.000, size=1500)
lngs = np.random.uniform(low=-180.000, high=180.000, size=1500)
lat_lngs = zip(lats, lngs)
lat_lngs
```
Next, I unpacked the lat_lngs zip object into a list:
```
# Add the latitudes and longitudes to a list.
coordinates = list(lat_lngs)
```
### *Generate Random World Cities*
To determine the nearest city using the coordinates in the lat_lngs tuple, I imported the Python’s citypy module and created a for loop to iterate through the coordinates’ zipped tuple and used the cityipy.nearest_city() with the latitude and longitude within the parentheses of and the city_name chained to the nearest_city() function. As cities were found, they were added to a list:
```
# Use the citipy module to determine city based on latitude and longitude.
from citipy import citipy
# Create a list for holding the cities.
cities = []
# Identify the nearest city for each latitude and longitude combination.
for coordinate in coordinates:
    city = citipy.nearest_city(coordinate[0], coordinate[1]).city_name

    # If the city is unique, then we will add it to the cities list.
    if city not in cities:
        cities.append(city)
# Print the city count to confirm sufficient count.
len(cities)
```
### *Get the City Weather Data*
For each city within my lat_lngs list, I used OpenWeatherMap API to retrieve weather-related data for each city from a JSON file.

To obtain weather data for each city, I began by importing my Requests Library and the weather_api_key:
```
# Import the requests library.
import requests

# Import the API key.
from config import weather_api_key
```
Then, I built the basic URL for the OpenWeatherMap with my weather_api_key added to the URL:
```
# Starting URL for Weather Map API Call.
url = "http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=" + weather_api_key
print(url)
```
Subsequently, I imported the datetime module:
```
# Import the datetime module from the datetime library.
from datetime import datetime
```
I declared an empty list to hold the weather data, added a print statement that referenced the beginning of the logging, created counters for the record numbers, 1-50, and set the counters to 1:
```
# Create an empty list to hold the weather data.
city_data = []
# Print the beginning of the logging.
print("Beginning Data Retrieval     ")
print("-----------------------------")

# Create counters.
record_count = 1
set_count = 1
```
Then, I created a for loop to iterate through the cities list and begin building the URL for each city. I grouped the cities into sets of 50 to log the progress. For the for loop, I used the enumerate() method to iterate through the list of cities and retrieve the index and city from the list to add to the city_url. Within the conditional statement, the URL endpoint was created for each city, with the blank spaces in the city name removed and the city name concatenated with city.replace(“ “, “+”) to ensure the weather data was found for the city and not the first part of the city name. A print statement was also added to state the record and set counts, as well as the city being processed. Record_count was then increased by 1 before the next city was processed:
```
# Loop through all the cities in the list.
for i, city in enumerate(cities):

    # Group cities in sets of 50 for logging purposes.
    if (i % 50 == 0 and i >= 50):
        set_count += 1
        record_count = 1
    # Create endpoint URL with each city.
    city_url = url + "&q=" + city.replace(" ","+")

    # Log the URL, record, and set numbers and the city.
    print(f"Processing Record {record_count} of Set {set_count} | {city}")
    # Add 1 to the record count.
    record_count += 1
```
I added a try-except block to my code to prevent the API request from stopping prematurely due to an invalid request. Below the try block, the code parsed the data from the JSON file, assigned variables for each piece of data needed (i.e., city, country, date, latitude, longitude, maximum temperature, humidity, cloudiness, and wind speed), and added the data to the cities list in a dictionary format. Below the except block, a print() statement was placed to indicate that a city was not found and was skipped. Another print() statement was added to inform that the data retrieval was complete:
```
# Run an API request for each of the cities.
    try:
        # Parse the JSON and retrieve data.
        city_weather = requests.get(city_url).json()
        # Parse out the needed data.
        city_lat = city_weather["coord"]["lat"]
        city_lng = city_weather["coord"]["lon"]
        city_max_temp = city_weather["main"]["temp_max"]
        city_humidity = city_weather["main"]["humidity"]
        city_clouds = city_weather["clouds"]["all"]
        city_wind = city_weather["wind"]["speed"]
        city_country = city_weather["sys"]["country"]
        # Convert the date to ISO standard.
        city_date = datetime.utcfromtimestamp(city_weather["dt"]).strftime('%Y-%m-%d %H:%M:%S')
        # Append the city information into city_data list.
        city_data.append({"City": city.title(),
                          "Lat": city_lat,
                          "Lng": city_lng,
                          "Max Temp": city_max_temp,
                          "Humidity": city_humidity,
                          "Cloudiness": city_clouds,
                          "Wind Speed": city_wind,
                          "Country": city_country,
                          "Date": city_date})

# If an error is experienced, skip the city.
    except:
        print("City not found. Skipping...")
        pass

# Indicate that Data Loading is complete.
print("-----------------------------")
print("Data Retrieval Complete      ")
print("-----------------------------")
```

### *Create a DataFrame of City Weather Data*
I converted the array of dictionaries to a DataFrame:
```
# Convert the array of dictionaries to a Pandas DataFrame.
city_data_df = pd.DataFrame(city_data)
city_data_df.head(10)
```
Then, I reordered the columns to the order of City, Country, Date, Lat, Lng, Max Temp, Humidity, Cloudiness, and Wind Speed:
```
# Reorder columns in city_data_df DataFrame.
new_column_order = ["City", "Country", "Date", "Lat", "Lng", "Max Temp", "Humidity", "Cloudiness", "Wind Speed"]

city_data_df = city_data_df[new_column_order]
city_data_df.head(10)
```
Finally, I created an output file to save the DataFrame as a CSV:
```
# Create the output file (CSV).
output_data_file = "weather_data/cities.csv"
# Export the City_Data into a CSV.
city_data_df.to_csv(output_data_file, index_label="City_ID")
```

### *Plot Latitude versus Temperature*
To create my scatter plots, I first added the following code to a new cell to gather the necessary data:
```
# Extract relevant fields from the DataFrame for plotting.
lats = city_data_df["Lat"]
max_temps = city_data_df["Max Temp"]
humidity = city_data_df["Humidity"]
cloudiness = city_data_df["Cloudiness"]
wind_speed = city_data_df["Wind Speed"]
```
The scatter plots needed to include the current date so I imported the title module and used the strftime() with the formatting parameters for the date in parentheses. The time.strftime(“%x”) was added to the plt.title() when making the scatter plot:
```
# Import time module
import time

# Build the scatter plot for latitude vs. max temperature.
plt.scatter(lats,
            max_temps,
            edgecolor="black", linewidths=1, marker="o",
            alpha=0.8, label="Cities")

# Incorporate the other graph properties.
plt.title(f"City Latitude vs. Max Temperature "+ time.strftime("%x"))
plt.ylabel("Max Temperature (F)")
plt.xlabel("Latitude")
plt.grid(True)

# Save the figure.
plt.savefig("weather_data/Fig1.png")

# Show plot.
plt.show()
```

![Fig1.png](https://github.com/kcharb7/World_Weather_Analysis/blob/main/weather_data/Fig1.png)

### *Plot Latitude versus Humidity*
Repurposing the code for the maximum temperature scatter plot, changing the y-axis variable to “humidity”, the title to “Humidity”, and the y-axis label to “Humidty(%)”, I create a scatter plot for latitude versus humidty:
```
# Build the scatter plots for latitude vs. humidity.
plt.scatter(lats,
            humidity,
            edgecolor="black", linewidths=1, marker="o",
            alpha=0.8, label="Cities")

# Incorporate the other graph properties.
plt.title(f"City Latitude vs. Humidity "+ time.strftime("%x"))
plt.ylabel("Humidity (%)")
plt.xlabel("Latitude")
plt.grid(True)
# Save the figure.
plt.savefig("weather_data/Fig2.png")
# Show plot.
plt.show()
```

![Fig2.png](https://github.com/kcharb7/World_Weather_Analysis/blob/main/weather_data/Fig2.png)

### *Plot Latitude versus Cloudiness*
Refactoring the code again by changing the y-axis variable to “cloudiness”, the title to “Cloudiness(%)”, and the y-axis label to “Cloudiness(%)”, I created a scatter plot for latitude versus cloudiness:
```
# Build the scatter plots for latitude vs. cloudiness.
plt.scatter(lats,
            cloudiness,
            edgecolor="black", linewidths=1, marker="o",
            alpha=0.8, label="Cities")

# Incorporate the other graph properties.
plt.title(f"City Latitude vs. Cloudiness (%) "+ time.strftime("%x"))
plt.ylabel("Cloudiness (%)")
plt.xlabel("Latitude")
plt.grid(True)
# Save the figure.
plt.savefig("weather_data/Fig3.png")
# Show plot.
plt.show()
```

![Fig3.png](https://github.com/kcharb7/World_Weather_Analysis/blob/main/weather_data/Fig3.png)

### *Plot Latitude versus Wind Speed*
Repurposing the code by changing the y-axis variable to “wind_speed”, the title to “Wind Speed”, and the y-axis label to “Wind Speed (mph)”, I created a scatter plot for latitude versus wind speed:
```
# Build the scatter plots for latitude vs. wind speed.
plt.scatter(lats,
            wind_speed,
            edgecolor="black", linewidths=1, marker="o",
            alpha=0.8, label="Cities")

# Incorporate the other graph properties.
plt.title(f"City Latitude vs. Wind Speed "+ time.strftime("%x"))
plt.ylabel("Wind Speed (mph)")
plt.xlabel("Latitude")
plt.grid(True)
# Save the figure.
plt.savefig("weather_data/Fig4.png")
# Show plot.
plt.show()
```

![Fig4.png](https://github.com/kcharb7/World_Weather_Analysis/blob/main/weather_data/Fig4.png)

### *Correlation Between Latitude and Maximum Temperature*
I imported the linear regression function from the SciPy statistics module so I could perform a linear regression and return the equation of the regression line, correlation coefficient (r-value), and p-value. This linear regression line was to be added to each of the scatter plots for latitude versus weather parameter. The basic code looked as follows:
```
# Import linear regression from the SciPy stats module.
from scipy.stats import linregress

# Perform linear regression.
(slope, intercept, r_value, p_value, std_err) = linregress(x_values, y_values)

# Calculate the regression line "y values" from the slope and intercept.
regress_values = x_values * slope + intercept

# Get the equation of the line.
line_eq = "y = " + str(round(slope,2)) + "x + " + str(round(intercept,2))

# Create a scatter plot of the x and y values.
plt.scatter(x_values,y_values)
# Plot the regression line with the x-values and the y coordinates based on the intercept and slope.
plt.plot(x_values,regress_values,"r")
# Annotate the text for the line equation and add its coordinates.
plt.annotate(line_eq, (10,40), fontsize=15, color="red")
plt.xlabel('Latitude')
plt.ylabel('Temp')
plt.show()
```
The code was to be reused for each of the weather parameters in both the Northern and Southern hemispheres, with minor changes. The x-value was the latitudes, the y values were each of the four weather parameters, the y label was the weather parameter being plotted, and the x- and y-values given as a tuple, (10,40), for the regression ling equation were to be placed on the scatter plot. 

As only a few changes to the code were needed to create each scatter plot, I converted the linear regression calculation and plotting into a function that included the four parameters as variables:
```
# Create a function to perform linear regression on the weather data and 
# plot a regression line and the equation with the data.
def plot_linear_regression(x_values, y_values, title, y_label, text_coordinates):
```
The algorithm used to perform the linear regression was added underneath the function:
```
# Import linregress
from scipy.stats import linregress

# Create a function to perform linear regression on the weather data and
# plot a regression line and the equation with the data.
def plot_linear_regression(x_values, y_values, title, y_label, text_coordinates):

    # Run regression on hemisphere weather data.
    (slope, intercept, r_value, p_value, std_err) = linregress(x_values, y_values)

    # Calculate the regression line "y values" from the slope and intercept.
    regress_values = x_values * slope + intercept
    # Get the equation of the line.
    line_eq = "y = " + str(round(slope,2)) + "x + " + str(round(intercept,2))
    # Create a scatter plot and plot the regression line.
    plt.scatter(x_values,y_values)
    plt.plot(x_values,regress_values,"r")
    # Annotate the text for the line equation.
    plt.annotate(line_eq, text_coordinates, fontsize=15, color="red")
    plt.xlabel('Latitude')
    plt.ylabel(y_label)
    plt.title(title)
    plt.show()
```
To perform regression analysis on weather parameters in the Northern and Southern Hemispheres, I created Northern and Southern Hemisphere DataFrames from the city_data_df DataFrame using the loc method. Within the brackets of the loc method, I included a conditional filter for latitudes greater than or equal to 0 to find only Northern Hemisphere latitudes. To obtain Southern Hemisphere latitudes, I used the same line of code and change the conditional filter to latitudes less than 0:
```
# Create Northern and Southern Hemisphere DataFrames.
northern_hemi_df = city_data_df.loc[(city_data_df["Lat"] >= 0)]
southern_hemi_df = city_data_df.loc[(city_data_df["Lat"] < 0)]
```
To generate the linear regression on the maximum temperature for the Northern Hemisphere, I set the x values equal to the latitude column and the y values equal to the maximum temperature column within the northern_hemi_df DataFrame. Then, I called the plot_linear_regression function and edited the title, y_label, and text_coordinates to refer to maximum temperature:
```
# Linear regression on the Northern Hemisphere
x_values = northern_hemi_df["Lat"]
y_values = northern_hemi_df["Max Temp"]
# Call the function.
plot_linear_regression(x_values, y_values,
                       'Linear Regression on the Northern Hemisphere for Maximum Temperature', 'Max Temp',(10,40))
```

![temp_north.png](https://github.com/kcharb7/World_Weather_Analysis/blob/main/weather_data/temp_north.png)

Using the following code, I determine the r-value to be -0.877, indicating a strong to very strong negative correlation between latitude and temperature in the Northern Hemisphere:
```
# Linear regression on the Northern Hemisphere
x_values = northern_hemi_df["Lat"]
y_values = northern_hemi_df["Max Temp"]
# Perform linear regression.
(slope, intercept, r_value, p_value, std_err) = linregress(x_values, y_values)
# Get the r-value
print(f"The r-value is: {r_value:.3f}")
```
The linear regression code was reused to generate the linear regression on maximum temperature for the Southern Hemisphere, replacing northern_hemi_df with southern_hemi_df:
```
# Linear regression on the Southern Hemisphere
x_values = southern_hemi_df["Lat"]
y_values = southern_hemi_df["Max Temp"]
# Call the function.
plot_linear_regression(x_values, y_values,
                       'Linear Regression on the Southern Hemisphere for Maximum Temperature', 'Max Temp',(-50,90))
```

![temp_south.png](https://github.com/kcharb7/World_Weather_Analysis/blob/main/weather_data/temp_south.png)

Using the following code, I determine the r-value to be 0.610, indicating a strong to very strong positive correlation between latitude and temperature in the Southern Hemisphere:
```
# Linear regression on the Southern Hemisphere
x_values = southern_hemi_df["Lat"]
y_values = southern_hemi_df["Max Temp"]
# Perform linear regression.
(slope, intercept, r_value, p_value, std_err) = linregress(x_values, y_values)
# Get the r-value
print(f"The r-value is: {r_value:.3f}")
```

### *Correlation Between Latitude and Percent Humidity*
To perform the linear regression on the percent humidity for the Northern Hemisphere, I set the x-value equal to the “Lat” column and the y-value equal to the “Humidity” column within the northern_hemi_df DataFrame. Then, I called the plot_linear_regression function with the x- and y-values and edited the title, y_label, and text_coordinated to refer to humidity:
```
# Linear regression on the Northern Hemisphere
x_values = northern_hemi_df["Lat"]
y_values = northern_hemi_df["Humidity"]
# Call the function.
plot_linear_regression(x_values, y_values,
                       'Linear Regression on the Northern Hemisphere for % Humidity', '% Humidity',(40,10))
```

![humid_north.png](https://github.com/kcharb7/World_Weather_Analysis/blob/main/weather_data/humid_north.png)

Using the following code, I determine the r-value to be 0.271, indicating a weak positive correlation between latitude and humidity in the Northern Hemisphere:
```
# Linear regression on the Northern Hemisphere
x_values = northern_hemi_df["Lat"]
y_values = northern_hemi_df["Humidity"]
# Perform linear regression.
(slope, intercept, r_value, p_value, std_err) = linregress(x_values, y_values)
# Get the r-value
print(f"The r-value is: {r_value:.3f}")
```
The linear regression code was reused to generate the linear regression on humidity for the Southern Hemisphere, replacing northern_hemi_df with southern_hemi_df:
```
# Linear regression on the Southern Hemisphere
x_values = southern_hemi_df["Lat"]
y_values = southern_hemi_df["Humidity"]
# Call the function.
plot_linear_regression(x_values, y_values,
                       'Linear Regression on the Southern Hemisphere for % Humidity', '% Humidity',(-50,15))
```

![humid_south.png](https://github.com/kcharb7/World_Weather_Analysis/blob/main/weather_data/humid_south.png)

Using the following code, I determine the r-value to be 0.282, indicating a weak positive correlation between latitude and humidity in the Southern Hemisphere:
```
# Linear regression on the Southern Hemisphere
x_values = southern_hemi_df["Lat"]
y_values = southern_hemi_df["Humidity"]
# Perform linear regression.
(slope, intercept, r_value, p_value, std_err) = linregress(x_values, y_values)
# Get the r-value
print(f"The r-value is: {r_value:.3f}")
```

### *Correlation Between Latitude and Percent Cloudiness*
Refactoring the code used for the previous linear regression lines and plots, I set the x-value equal to the “Lat” column and the y-value equal to the “Cloudiness” column within the northern_hemi_df DataFrame. Then, I called the plot_linear_regression function with the x- and y-values and edited the title, y_label, and text_coordinated to refer to cloudiness:
```
# Linear regression on the Northern Hemisphere
x_values = northern_hemi_df["Lat"]
y_values = northern_hemi_df["Cloudiness"]
# Call the function.
plot_linear_regression(x_values, y_values,
                       'Linear Regression on the Northern Hemisphere for % Cloudiness', '% Cloudiness',(10,60))
```

![cloud_north.png](https://github.com/kcharb7/World_Weather_Analysis/blob/main/weather_data/cloud_north.png)

Then, I determined the r-value using the following code:
```
# Linear regression on the Northern Hemisphere
x_values = northern_hemi_df["Lat"]
y_values = northern_hemi_df["Cloudiness"]
# Perform linear regression.
(slope, intercept, r_value, p_value, std_err) = linregress(x_values, y_values)
# Get the r-value
print(f"The r-value is: {r_value:.3f}")
```
The r-value was 0.235, indicating a weak positive correlation between latitude and cloudiness in the Northern Hemisphere.

Next, I refactored my linear regression and plots code to create the plot for the Southern Hemisphere:
```
# Linear regression on the Southern Hemisphere
x_values = southern_hemi_df["Lat"]
y_values = southern_hemi_df["Cloudiness"]
# Call the function.
plot_linear_regression(x_values, y_values,
                       'Linear Regression on the Southern Hemisphere for % Cloudiness', '% Cloudiness',(-50,60))
```

![cloud_south.png](https://github.com/kcharb7/World_Weather_Analysis/blob/main/weather_data/cloud_south.png)

Refactoring the code from above, I determined the r-value:
```
# Linear regression on the Southern Hemisphere
x_values = southern_hemi_df["Lat"]
y_values = southern_hemi_df["Cloudiness"]
# Perform linear regression.
(slope, intercept, r_value, p_value, std_err) = linregress(x_values, y_values)
# Get the r-value
print(f"The r-value is: {r_value:.3f}")
```

The r-value was 0.213, indicating a weak positive correlation between latitude and cloudiness in the Southern Hemisphere. 

### *Correlation Between Latitude and Wind Speed*
Refactoring the code used for the previous linear regression lines and plots, I set the x-value equal to the “Lat” column and the y-value equal to the “Wind Speed” column within the northern_hemi_df DataFrame. Then, I called the plot_linear_regression function with the x- and y-values and edited the title, y_label, and text_coordinated to refer to wind speed:
```
# Linear regression on the Northern Hemisphere
x_values = northern_hemi_df["Lat"]
y_values = northern_hemi_df["Wind Speed"]
# Call the function.
plot_linear_regression(x_values, y_values,
                       'Linear Regression on the Northern Hemisphere for Wind Speed', 'Wind Speed',(40,35))
```

![wind_north.png](https://github.com/kcharb7/World_Weather_Analysis/blob/main/weather_data/wind_north.png)

Using the following code, I determined the r-value to be 0.108, indicating a weak correlation between latitude and wind speed in the Northern Hemisphere:
```
# Linear regression on the Northern Hemisphere
x_values = northern_hemi_df["Lat"]
y_values = northern_hemi_df["Wind Speed"]
# Perform linear regression.
(slope, intercept, r_value, p_value, std_err) = linregress(x_values, y_values)
# Get the r-value
print(f"The r-value is: {r_value:.3f}")
```
Next, I refactored my linear regression and plots code to create the plot for the Southern Hemisphere:
```
# Linear regression on the Southern Hemisphere
x_values = southern_hemi_df["Lat"]
y_values = southern_hemi_df["Wind Speed"]
# Call the function.
plot_linear_regression(x_values, y_values,
                       'Linear Regression on the Southern Hemisphere for Wind Speed', 'Wind Speed',(-50,35))
```

![wind_south.png](https://github.com/kcharb7/World_Weather_Analysis/blob/main/weather_data/wind_south.png)

Using the following code, I determined the r-value to be -0.334, indicating a weak negative correlation between latitude and wind speed in the Southern Hemisphere:
```
# Linear regression on the Southern Hemisphere
x_values = southern_hemi_df["Lat"]
y_values = southern_hemi_df["Wind Speed"]
# Perform linear regression.
(slope, intercept, r_value, p_value, std_err) = linregress(x_values, y_values)
# Get the r-value
print(f"The r-value is: {r_value:.3f}")
```

### *Heatmaps for Weather Parameters*
To create heatmaps for each of the weather parameters, I first imported my dependencies and Google API key into a new Jupyter Notebook file VacationPy.ipynb:
```
# Import the dependencies.
import pandas as pd
import gmaps
import requests
# Import the API key.
from config import g_key
```
Then, I configured gmaps to use my Google API key:
```
# Configure gmaps to use your Google API key.
gmaps.configure(api_key=g_key)
```
I created a maximum temperature heatmap using the general syntax for generating a heatmap and adding the “Lat” and “Lng” columns to the locations array and the values from the “Max temp” column of the city_data_df DataFrame. As Google heatmaps do not plot negative numbers, I additionally performed a list comprehension within the heat_layer() function to add temperature greater than 0 degrees Fahrenheit. Then, I adjusted the heatmap zoom, intensity, and point radius:
```
# Heatmap of temperature
# Get the latitude and longitude.
locations = city_data_df[["Lat", "Lng"]]
# Get the maximum temperature.
max_temp = city_data_df["Max Temp"]
# Assign the figure variable.
fig = gmaps.figure(center=(30.0, 31.0), zoom_level=1.5)
# Assign the heatmap variable.
heat_layer = gmaps.heatmap_layer(locations, weights=[max(temp, 0) for temp in max_temp], dissipating=False, max_intensity=300, point_radius=4)
# Add the heatmap layer.
fig.add_layer(heat_layer)
# Call the figure to plot the data.
fig
```
The code was refactored to create a percent humidity heatmap:
```
# Heatmap of percent humidity
locations = city_data_df[["Lat", "Lng"]]
humidity = city_data_df["Humidity"]
fig = gmaps.figure(center=(30.0, 31.0), zoom_level=1.5)
heat_layer = gmaps.heatmap_layer(locations, weights=humidity, dissipating=False, max_intensity=300, point_radius=4)

fig.add_layer(heat_layer)
# Call the figure to plot the data.
fig
```
Again, the code was refactored to make a percent cloudiness heatmap:
```
# Heatmap of percent cloudiness
locations = city_data_df[["Lat", "Lng"]]
clouds = city_data_df["Cloudiness"]
fig = gmaps.figure(center=(30.0, 31.0), zoom_level=1.5)
heat_layer = gmaps.heatmap_layer(locations, weights=clouds, dissipating=False, max_intensity=300, point_radius=4)

fig.add_layer(heat_layer)
# Call the figure to plot the data.
fig
```
Finally, I refactored the code to make a wind speed heatmap:
```
# Heatmap of wind speed
locations = city_data_df[["Lat", "Lng"]]
wind = city_data_df["Wind Speed"]
fig = gmaps.figure(center=(30.0, 31.0), zoom_level=1.5)
heat_layer = gmaps.heatmap_layer(locations, weights=wind, dissipating=False, max_intensity=300, point_radius=4)

fig.add_layer(heat_layer)
# Call the figure to plot the data.
fig
```

### *Vacation Criteria*
For the app being created, the users needed to be prompted to enter their minimum and maximum temperature ranges as floating-point decimal numbers to filter the city_data_df DataFrame. To do this, I wrapped the input() statement with the float() method and created two input statements for the app to prompt customers to add minimum and maximum temperatures:
```
# Ask the customer to add a minimum and maximum temperature value.
min_temp = float(input("What is the minimum temperature you would like for your trip? "))
max_temp = float(input("What is the maximum temperature you would like for your trip? "))
```
Then, I used logical operators to filter the “Max Temp” column within the city_data_df DataFrame and create a new DataFrame with the cities that meet the customer’s criteria:
```
# Filter the dataset to find the cities that fit the criteria.
preferred_cities_df = city_data_df.loc[(city_data_df["Max Temp"] <= max_temp) & \
                                       (city_data_df["Max Temp"] >= min_temp)]
preferred_cities_df.head(10)
```
Before moving on, I checked whether the preferred_cities_df DataFrame had any null values using the following code:
```
preferred_cities_df.count()
```
The DataFrame contained 185 cities and there were no null values. 

### *Map Vacation Criteria*
Once customers have filtered the data based on their preferred temperatures, a heatmap with maximum temperatures for the filtered cities needed to be made, as well as a marker for each city that when clicked would display the name of the city, country code, maximum temperature, and name of a hotel within 3 miles of the coordinates. To find a nearby hotel, I used the coordinates from the preferred_cities_df and Google Places API, placing the retrieved hotel information into a new DataFrame. This new DataFrame, hotel_df, was created by making a copy of the prefferred_cities_df, keeping the columns “City”, “Country”, “Max Temp”, “Lat”, and “Lng”, and adding the new column “Hotel Name”:
```
# Create DataFrame called hotel_df to store hotel names along with city, country, max temp, and coordinates.
hotel_df = preferred_cities_df[["City", "Country", "Max Temp", "Lat", "Lng"]].copy()
hotel_df["Hotel Name"] = ""
hotel_df.head(10)
```
To retrieve nearby hotels, I used the Google Placed Nearby Search request with the following parameters:
-	API key
-	Latitude and longitude
-	5,000-meter radius
-	Type of place (i.e., lodging)

These parameters were added as key-value pairs in a params dictionary:
```
# Set parameters to search for a hotel.
params = {
    "radius": 5000,
    "type": "lodging",
    "key": g_key
}
```
Then, I used the iterrows() function to iterate through the hotel_df DataFrame to get the latitude and longitude  values and add them to the params dictionary. Then the params were added along with the base URL to search for lodging and request the JSON data. Then, a try-except block was added to handle any errors while retrieving the hotel name and storing it in the hotel_df DataFrame:
```
 # Iterate through the DataFrame.
for index, row in hotel_df.iterrows():
    # Get the latitude and longitude.
    lat = row["Lat"]
    lng = row["Lng"]

    # Add the latitude and longitude to location key for the params dictionary.
    params["location"] = f"{lat},{lng}"

    # Use the search term: "lodging" and our latitude and longitude.
    base_url = "https://maps.googleapis.com/maps/api/place/nearbysearch/json"
    # Make request and get the JSON data from the search.
    hotels = requests.get(base_url, params=params).json()
    # Grab the first hotel from the results and store the name.
    try:
        hotel_df.loc[index, "Hotel Name"] = hotels["results"][0]["name"]
    except (IndexError):
        print("Hotel not found... skipping.")
```
The code used for the prior maximum temperature heatmap was refactored to create a heatmap of the maximum temperatures within the hotel_df DataFrame:
```
# Add a heatmap of temperature for the vacation spots.
locations = hotel_df[["Lat", "Lng"]]
max_temp = hotel_df["Max Temp"]
fig = gmaps.figure(center=(30.0, 31.0), zoom_level=1.5)
heat_layer = gmaps.heatmap_layer(locations, weights=max_temp, dissipating=False,
             max_intensity=300, point_radius=4)

fig.add_layer(heat_layer)
# Call the figure to plot the data.
fig
```
I edited the above code to add markers for each city on top of the heatmap using the markers = gmaps.marker_layer(marker_locations), with the marker_locations as the latitudes and longitudes:
```
# Add a heatmap of temperature for the vacation spots and marker for each city.
locations = hotel_df[["Lat", "Lng"]]
max_temp = hotel_df["Max Temp"]
fig = gmaps.figure(center=(30.0, 31.0), zoom_level=1.5)
heat_layer = gmaps.heatmap_layer(locations, weights=max_temp,
             dissipating=False, max_intensity=300, point_radius=4)
marker_layer = gmaps.marker_layer(locations)
fig.add_layer(heat_layer)
fig.add_layer(marker_layer)
# Call the figure to plot the data.
fig
```
To add a pop-up marker for each city that displayed the hotel name, city name, country, and maximum temperature, I added an info_box_template with the following syntax:
```
info_box_template = """
<dl>
<dt>Hotel Name</dt><dd>{Hotel Name}</dd>
<dt>City</dt><dd>{City}</dd>
<dt>Country</dt><dd>{Country}</dd>
<dt>Max Temp</dt><dd>{Max Temp} °F</dd>
</dl>
"""
```
Then, the data was added to the code by iterating through the hotel_df DataFrame using the iterrows() function:
```
# Store the DataFrame Row.
hotel_info = [info_box_template.format(**row) for index, row in hotel_df.iterrows()]
```
The info_box_content=hotel_info was added alongside the locations to the gmaps.marker_layer() attribute within the code used to create the heatmap with markers:
```
# Add a heatmap of temperature for the vacation spots and a pop-up marker for each city.
locations = hotel_df[["Lat", "Lng"]]
max_temp = hotel_df["Max Temp"]
fig = gmaps.figure(center=(30.0, 31.0), zoom_level=1.5)
heat_layer = gmaps.heatmap_layer(locations, weights=max_temp,dissipating=False,
             max_intensity=300, point_radius=4)
marker_layer = gmaps.marker_layer(locations, info_box_content=hotel_info)
fig.add_layer(heat_layer)
fig.add_layer(marker_layer)

# Call the figure to plot the data.
fig
```

# WeatherPy Challenge
## Overview
### *Purpose*
The PlanMyTrip app was a hit with Jack and the Beta testers, however, they provided some recommendations to further improve the app. Specifically, they recommended the addition of the weather description to the weather data previously retrieved so that the weather data can be filtered through input statements based on a customer’s weather preferences. These weather preferences will then be used to identify potential travel destinations and nearby hotels. The customers should then be able to select four cities from the list of potential destinations to create a travel itinerary. A travel route as well as a marker layer map will be created for customers using Google Maps Directions API. 

## Deliverables
### *Retrieve Weather Data*
To begin, I created a new folder called Weather_Database and within the folder I created a new Jupyter Notebook file Weather_Database.ipynb.

Within the Weather_Database.ipynb file, I imported my dependencies:
```
# Import the dependencies.
import pandas as pd
import numpy as np
```
Chaining the NumPy module to the random module, I created code to generate 2,000 random latitudes and longitudes as floating-point decimal numbers. Then I used the zip() function to pack the latitudes and longitudes as pairs:
```
# Create a set of random latitude and longitude combinations.
lats = np.random.uniform(low=-90.000, high=90.000, size=2000)
lngs = np.random.uniform(low=-180.000, high=180.000, size=2000)
lat_lngs = zip(lats, lngs)
```
Then, I added the latitudes and longitudes to a list:
```
# Add the latitudes and longitudes to a list.
coordinates = list(lat_lngs)
```
To determine the nearest city for each latitude and longitude combination I used the citipy module. I created a for loop to iterate through the coordinates’ zipped tuple. Within the for loop I used the cityipy.nearest_city() with the latitude and longitude within the parentheses and chained the city_name to the nearest_city() function to find the nearest city. As cities were found, they were added to a list:
```

# Use the citipy module to determine city based on latitude and longitude.
from citipy import citipy

# Create a list for holding the cities.
cities = []

#Identify the nearest city for each latitude and longitude combination.
for coordinate in coordinates:
    city = citipy.nearest_city(coordinate[0], coordinate[1]).city_name
    
    # If the city is unique, then add it to the cities list. 
    if city not in cities:
        cities.append(city)
        
# Print the city count to confirm sufficient count.
len(cities)
```
To obtain weather data for each city, I imported my Requests Library and the weather_api_key:
```
# Import the requests library.
import requests

# Import the API key.
from config import weather_api_key
```
Then, I built the basic URL for the OpenWeatherMap with my weather_api_key added to the URL:
```
# Starting URL for Weather Map API Call.
url = "http://api.openweathermap.org/data/2.5/weather?units=Imperial&APPID=" + weather_api_key
print(url)
```
As was done previously to retrieve the weather data, I declared an empty list to hold the weather data, added a print statement that referenced the beginning of the logging, created counters for the record numbers, 1-50, and set the counters to 1. Then, I created a for loop to iterate through the cities list and begin building the URL for each city. I grouped the cities into sets of 50 to log the progress. For the for loop, I used the enumerate() method to iterate through the list of cities and retrieve the index and city from the list to add to the city_url. Within the conditional statement, the URL endpoint was created for each city, with the blank spaces in the city name removed and the city name concatenated with city.replace(“ “, “+”) to ensure the weather data was found for the city and not the first part of the city name. A print statement was also added to state the record and set counts, as well as the city being processed. Record_count was then increased by 1 before the next city was processed. I added a try-except block to my code to prevent the API request from stopping prematurely due to an invalid request. Below the try block, the code parsed the data from the JSON file, assigned variables for each piece of data needed (i.e., city, country, latitude, longitude, maximum temperature, humidity, cloudiness, wind speed, and current description), and added the data to the cities list in a dictionary format. Below the except block, a print() statement was placed to indicate if a city was not found and was skipped. Another print() statement was added to inform that the data retrieval was complete. The final code look as follows:
```
# Create an empty list to hold the weather data.
city_data = []

# Print the beginning of the logging.
print("Beginning Data Retrieval     ")
print("-----------------------------")

# Create counters.
record_count = 1
set_count = 1

# Loop through all the cities in the list.
for i, city in enumerate(cities):
    
    # Group cities in sets of 50 for logging purposes.
    if (i % 50 == 0 and i >= 50):
        set_count +=1
        record_count = 1
        
    # Create endpoint URL with each city.
    city_url = url + "&q=" + city.replace(" ", "+")
    
    # Log the URL, record, and set numbers and the city.
    print(f"Processing Record {record_count} of Set {set_count} | {city}")
    
    # Add 1 to the record count. 
    record_count += 1

# Run an API request for each of the cities.
    try: 
        # Parse the JSON and retrieve data. 
        city_weather = requests.get(city_url).json()
        
        #Parse out the needed data.
        city_lat = city_weather["coord"]["lat"]
        city_lng = city_weather["coord"]["lon"]
        city_max_temp = city_weather["main"]["temp_max"]
        city_humidity = city_weather["main"]["humidity"]
        city_clouds = city_weather["clouds"]["all"]
        city_wind = city_weather["wind"]["speed"]
        city_country = city_weather["sys"]["country"]
        city_description = city_weather["weather"][0]["description"]
        
        # Append the city information into city_data_list.
        city_data.append({"City": city.title(),
                          "Lat": city_lat,
                          "Lng": city_lng, 
                          "Max Temp": city_max_temp,
                          "Humidity": city_humidity, 
                          "Cloudiness": city_clouds, 
                          "Wind Speed": city_wind, 
                          "Country": city_country, 
                          "Current Description": city_description})

# If an error is experienced, skip the city.
    except:
        print("City not found. Skipping...")
        pass

# Indicate the Data Loading is complete.
print("-----------------------------")
print("Data Retrieval Complete      ")
print("-----------------------------")
```
Once the weather data was retrieved, I converted the array of dictionaries to a DataFrame:
```
# Convert the array of dictionaries to a Pandas DataFrame.
city_data_df = pd.DataFrame(city_data)
city_data_df.head(10)
```
Then, I reordered the columns of the city_data_df DataFrame:
```
# Reorder columns in city_data_df DataFrame.
new_column_order = ["City", "Country", "Lat", "Lng", "Max Temp", "Humidity", "Cloudiness", "Wind Speed", "Current Description"]

city_data_df = city_data_df[new_column_order]
city_data_df.head(10)
```
Once the DataFrame was complete, I exported it as a CSV file and saved it in the Weather_Database folder:
```
# Create the output file (CSV).
output_data_file = "WeatherPy_Database.csv"

# Export the City_data into a CSV.
city_data_df.to_csv(output_data_file, index_label="City_ID")
```

### *Create a Customer Travel Destinations Map*
I created a new folder called Vacation_Search, then downloaded and renamed the starter code Vacation_Search.ipynb. Within the starter code I imported my dependencies and API keys:
```
# Dependencies and Setup
import pandas as pd
import requests
import gmaps

# Import API key
from config import g_key

# Configure gmaps API key
gmaps.configure(api_key=g_key)
```
Next, I imported the WeaterPy_Database.csv file:
```
# Import the WeatherPy_database.csv file. 
city_data_df = pd.read_csv("../Weather_Database/WeatherPy_database.csv")
city_data_df.head()
```
Within a new cell, I created two input statements that prompt the user to enter their minimum and maximum temperature criteria for their vacation:
```
# Prompt the user to enter minimum and maximum temperature criteria 
min_temp = float(input("What is the minimum temperature you would like for your trip? "))
max_temp = float(input("What is the maximum temperature you would like for your trip? "))
```
The loc method was then used to filter the city_data_df DataFrame and create a new DataFrame based on the temperature criteria inputted:
```
# Filter the city_data_df DataFrame using the input statements to create a new DataFrame using the loc method.
preferred_cities_df = city_data_df.loc[(city_data_df["Max Temp"] <= max_temp) & \
                                       (city_data_df["Max Temp"] >= min_temp)]
preferred_cities_df.head(10)
```
Once the new DataFrame was produced, I used the isnull() and sum() methods to determine if there were any empty rows within the DataFrame:
```
# Determine if there are any empty rows.
preferred_cities_df.isnull().sum()
```
The output indicated there were 3 empty rows within the “Country” column. To remove these empty rows, I used the dropna() method:
```
# Drop any empty rows and create a new DataFrame that doesn’t have empty rows.
clean_df = preferred_cities_df.dropna()

clean_df.head(10)
```
Once the DataFrame was cleaned I created a new DataFrame, hotel_df, to hold the hotel names from the hotel search:
```
# Create DataFrame called hotel_df to store hotel names along with city, country, max temp, and coordinates.
hotel_df = clean_df[["City", "Country", "Max Temp", "Current Description", "Lat", "Lng"]].copy()

# Create a new column "Hotel Name"
hotel_df["Hotel Name"] = ""
hotel_df.head(10)
```
I set the search parameters to the same as my prior hotel search. The code was set to iterate through the hotel_df DataFrame to retrieve the latitude and longitude of each city to find the nearest hotel and then add the hotel name to the hotel_df DataFrame or skip to the next city if a hotel wasn’t found:
```
# Set parameters to search for hotels within 5000 meters.
params = {
    "radius": 5000,
    "type": "lodging",
    "key": g_key
}

# Iterate through the hotel DataFrame.
for index, row in hotel_df.iterrows():
    # Get latitude and longitude from DataFrame
    lat = row["Lat"]
    lng = row["Lng"]
    
    # Add the latitude and longitude to location key for the params dictionary.
    params["location"] = f"{lat},{lng}"
    
    # Set up the base URL for the Google Directions API to get JSON data.
    base_url = "https://maps.googleapis.com/maps/api/place/nearbysearch/json"

    # Make request and retrieve the JSON data from the search. 
    hotels = requests.get(base_url, params=params).json()
    
    # Get the first hotel from the results and store the name, if a hotel isn't found skip the city.
    try:
        hotel_df.loc[index, "Hotel Name"] = hotels["results"][0]["name"]
    except (IndexError):
        print("Hotel not found... skipping.")   
```
After the hotel_df DataFrame was created, I dropped any rows where no hotel was found:
```
# Drop any rows where there is no Hotel Name.
clean_hotel_df = hotel_df.dropna()

clean_hotel_df.head(10)
```
Then, I proceeded to save the DataFrame:
```
# Create the output File (CSV)
output_data_file = "WeatherPy_vacation.csv"
# Export the City_Data into a csv
clean_hotel_df.to_csv(output_data_file, index_label="City_ID")
```
After saving the DataFrame to a csv file, I added the city name, country code, the weather description, and the maximum temperature for the city to the info_box_template format. Then I used the provided list comprehension code to retrieve the city data from each row, which was then added to the formatting template and saved in the hotel_info list. I additionally retrieved the latitude and longitude from each row and stored them in a new DataFrame:
```
# Using the template add city name, the country code, the weather description and maximum temperature for the city.
info_box_template = """
<dl>
<dt>Hotel Name</dt><dd>{Hotel Name}</dd>
<dt>City</dt><dd>{City}</dd>
<dt>Country</dt><dd>{Country}</dd>
<dt>Weather Description</dt><dd>{Current Description} and {Max Temp} °F</dd>
</dl>
"""

# Get the data from each row and add it to the formatting template and store the data in a list.
hotel_info = [info_box_template.format(**row) for index, row in clean_hotel_df.iterrows()]

# Get the latitude and longitude from each row and store in a new DataFrame.
locations = clean_hotel_df[["Lat", "Lng"]]
```
Finally, I refactored my previous marker layer map code to create a marker layer map with pop-up markers for each city on the map:
```
# Add a marker layer for each city to the map. 
fig = gmaps.figure(center=(30.0, 31.0), zoom_level=1.5)
marker_layer = gmaps.marker_layer(locations, info_box_content=hotel_info)
fig.add_layer(marker_layer)

# Display the figure
fig
```

![WeatherPy_vacation_map.png](https://github.com/kcharb7/World_Weather_Analysis/blob/main/Vacation_Search/WeatherPy_vacation_map.png)

### *Create a Travel Itinerary*
To create the travel itinerary, I created a new folder called Vacation_Itinerary and downloaded the Vacation_Itinerary_starter_code.ipynb file, renaming it Vacation_Itinerary.ipynb. Once the file was opened, I imported my dependencies and API keys:
```
# Dependencies and Setup
import pandas as pd
import requests
import gmaps

# Import API key
from config import g_key

# Configure gmaps
gmaps.configure(api_key=g_key)
```
Then, I imported the WeatherPy_vacation.csv file:
```
# Read the WeatherPy_vacation.csv into a DataFrame.
vacation_df = pd.read_csv("../Vacation_Search/WeatherPy_vacation.csv")
vacation_df.head()
```
I refactored the code used for the previous marker layer map to create a marker layer map of the vacation search results:
```
# Using the template add city name, the country code, the weather description and maximum temperature for the city.
info_box_template = """
<dl>
<dt>Hotel Name</dt><dd>{Hotel Name}</dd>
<dt>City</dt><dd>{City}</dd>
<dt>Country</dt><dd>{Country}</dd>
<dt>Weather Description</dt><dd>{Current Description} and {Max Temp} °F</dd>
</dl>
"""

# Get the data from each row and add it to the formatting template and store the data in a list.
hotel_info = [info_box_template.format(**row) for index, row in vacation_df.iterrows()]

# Get the latitude and longitude from each row and store in a new DataFrame.
locations = vacation_df[["Lat", "Lng"]]

# Add a marker layer for each city to the map. 
fig = gmaps.figure(center=(30.0, 31.0), zoom_level=1.5)
marker_layer = gmaps.marker_layer(locations, info_box_content=hotel_info)
fig.add_layer(marker_layer)

# Display the figure
fig
```
Looking at the marker layer map, I selected four cities within a country that a customer may want to visit and used those city names and the loc method to create separate DataFrames for each city:
```
# From the map above pick 4 cities and create a vacation itinerary route to travel between the four cities. 
# Create DataFrames for each city by filtering the 'vacation_df' using the loc method. 
vacation_start = vacation_df.loc[(vacation_df["City"]=="Cacapava Do Sul")]
vacation_end = vacation_df.loc[(vacation_df["City"]=="Cacapava Do Sul")]
vacation_stop1 = vacation_df.loc[(vacation_df["City"]=="Arroio Dos Ratos")]
vacation_stop2 = vacation_df.loc[(vacation_df["City"]=="Cidreira")] 
vacation_stop3 = vacation_df.loc[(vacation_df["City"]=="Santa Maria")]
```
Using the to_numpy() function with list indexing, I wrote code to retrieve the latitude-longitude pairs as tuples from each city DataFrame:
```
# Get the latitude-longitude pairs as tuples from each city DataFrame using the to_numpy function and list indexing.
start = vacation_start[["Lat", "Lng"]].to_numpy()[0]
end = vacation_end[["Lat", "Lng"]].to_numpy()[0]
stop1 = vacation_stop1[["Lat", "Lng"]].to_numpy()[0]
stop2 = vacation_stop2[["Lat", "Lng"]].to_numpy()[0]
stop3 = vacation_stop3[["Lat", "Lng"]].to_numpy()[0]
```
After gathering the latitude and longitude pairs for each city, I created a directions layer map:
```
# Create a direction layer map using the start and end latitude-longitude pairs,
# and stop1, stop2, and stop3 as the waypoints. The travel_mode should be "DRIVING", "BICYCLING", or "WALKING".
fig = gmaps.figure()
directions = gmaps.directions_layer(start, end, waypoints=[stop1,stop2,stop3], travel_mode="DRIVING"or"BICYCLING"or"WALKING")
fig.add_layer(directions)
fig
```

![WeatherPy_travel_map.png](https://github.com/kcharb7/World_Weather_Analysis/blob/main/Vacation_Itinerary/WeatherPy_travel_map.png)

Using the concat() function, I combined the four separate city DataFrame into one DataFrame:
```
# To create a marker layer map between the four cities.
#  Combine the four city DataFrames into one DataFrame using the concat() function.
itinerary_df = pd.concat([vacation_start,vacation_stop1, vacation_stop2, vacation_stop3],ignore_index=True)
itinerary_df
```
Then, I refactored my previous code for making the marker layer map to create a map of the cities on the travel route:
```
# Using the template add city name, the country code, the weather description and maximum temperature for the city. 
info_box_template = """
<dl>
<dt>Hotel Name</dt><dd>{Hotel Name}</dd>
<dt>City</dt><dd>{City}</dd>
<dt>Country</dt><dd>{Country}</dd>
<dt>Weather Description</dt><dd>{Current Description} and {Max Temp} °F</dd>
</dl>
"""

# Get the data from each row and add it to the formatting template and store the data in a list.
hotel_info = [info_box_template.format(**row) for index, row in itinerary_df.iterrows()]

# Get the latitude and longitude from each row and store in a new DataFrame.
locations = itinerary_df[["Lat", "Lng"]]

# Add a marker layer for each city to the map. 
fig = gmaps.figure(center=(30.0, 31.0), zoom_level=1.5)
marker_layer = gmaps.marker_layer(locations, info_box_content=hotel_info)
fig.add_layer(marker_layer)

# Display the figure
fig
```

![WeatherPy_travel_map_markers.png](https://github.com/kcharb7/World_Weather_Analysis/blob/main/Vacation_Itinerary/WeatherPy_travel_map_markers.png)
