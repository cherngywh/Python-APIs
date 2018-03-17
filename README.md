# WeatherPy - Analyze Worldwide weather with OpenWeatherMap API  

## Analysis
*- The highest temperature (100F) happens at latitude 0, it's a beautiful curve. the farer from latitude 0, the lower of max tempture.*</br>
*- There is no significant relationship between latitude and cloudiness.*</br>
*- Even though we see some cities with lower humidity and higher wind speed between latitude 0~40, we still can find a tred to show a significant relationship.*


```python
import requests as req
import json
import datetime
import random
import time
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.dates import DateFormatter
from citipy import citipy
```

## Generate Cities List


```python
filename = 'apikey.txt'
```


```python
def get_file_contents(filename):
    try:
        with open(filename, 'r') as f:
            return f.read().strip()
    except FileNotFoundError:
        print("'%s' file not found" % filename)
```


```python
# Set up the Link
api_key = get_file_contents(filename)
url = 'http://api.openweathermap.org/data/2.5/weather?'
unit = 'Imperial'
Link = url + 'appid=' + api_key + '&units=' + unit + '&q='

# Generate lists of latitude and longtitude
lats = np.random.uniform(-90, 90, 3000)
lngs = np.random.uniform(-180, 180, 3000)

# Find the nearest city by citipy and then clean and shuffle the data
coords = []
cities = []
weather_data = []
for i in range(len(lats)):
    x = (lats[i], lngs[i])
    coords.append(x)
    city = citipy.nearest_city(x[0],x[1]).city_name
    cities.append(city)
    cities = list(set(cities))
    random.shuffle(cities)
len(cities)
```




    971




```python
# Create a function to get data from openweathermap, and filter out the cities which are not in the database
def city_query(sample_cities):
    for j in sample_cities:
        response = req.get(Link + j).json()
        if response['cod'] != '404':
            weather_data.append(response)
    return weather_data

# Create a timer to avoid exceeding limit per minute
all_cities = []
count = 0

for i in range(0, len(cities), 20):
    epoch = cities[i:i+20]
    all_cities += city_query(epoch)  
    count += 1
    print(count)
    print(len(all_cities))
    if len(all_cities) > 500:
        break
    time.sleep(120)
print('end')
```

    1
    19
    2
    55
    3
    110
    4
    183
    5
    273
    6
    380
    7
    506
    end



```python
# Get the data from json
Name = [data['name'] for data in all_cities]
Humidity = [data['main']['humidity'] for data in all_cities]
Lat = [data['coord']['lat'] for data in all_cities]
Lon = [data['coord']['lon'] for data in all_cities]
Cloudiness = [data['clouds']['all'] for data in all_cities]
Wind_Speed = [data['wind']['speed'] for data in all_cities]
Country = [data['sys']['country'] for data in all_cities]
Max_Temp = [data['main']['temp_max'] for data in all_cities]
Date = [data['dt'] for data in all_cities]

# Create a DataFrame
Weather_df = pd.DataFrame({'City':Name,
              'Cloudiness':Cloudiness,
              'Country':Country,
              'Date':Date,
              'Humidity':Humidity,
              'Lat':Lat,
              'Lng':Lon,
              'Max Temp':Max_Temp,
              'Wind Speed':Wind_Speed
             })
Weather_df.count()
```




    City          506
    Cloudiness    506
    Country       506
    Date          506
    Humidity      506
    Lat           506
    Lng           506
    Max Temp      506
    Wind Speed    506
    dtype: int64




```python
# Show DataFrame
Weather_df.to_csv('city_data.csv')
Weather_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Cloudiness</th>
      <th>Country</th>
      <th>Date</th>
      <th>Humidity</th>
      <th>Lat</th>
      <th>Lng</th>
      <th>Max Temp</th>
      <th>Wind Speed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Sri Aman</td>
      <td>90</td>
      <td>MY</td>
      <td>1514062800</td>
      <td>94</td>
      <td>1.24</td>
      <td>111.46</td>
      <td>75.20</td>
      <td>0.81</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Labuhan</td>
      <td>44</td>
      <td>ID</td>
      <td>1514066819</td>
      <td>91</td>
      <td>-2.54</td>
      <td>115.51</td>
      <td>74.87</td>
      <td>3.60</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Portland</td>
      <td>75</td>
      <td>US</td>
      <td>1514065980</td>
      <td>60</td>
      <td>45.52</td>
      <td>-122.67</td>
      <td>41.00</td>
      <td>20.80</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Ocos</td>
      <td>20</td>
      <td>GT</td>
      <td>1514062800</td>
      <td>41</td>
      <td>14.51</td>
      <td>-92.19</td>
      <td>93.20</td>
      <td>9.17</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Halifax</td>
      <td>90</td>
      <td>CA</td>
      <td>1514064600</td>
      <td>100</td>
      <td>44.65</td>
      <td>-63.58</td>
      <td>37.40</td>
      <td>19.46</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Transfer the date to readable format
data_date = time.strftime('(%m/%d/%y)', time.localtime(Weather_df['Date'][0]))
```

## Latitude vs Temperature Plot


```python
# Create a scatter plot to show latitude and max temperature
plt.scatter(Weather_df['Lat'], Weather_df['Max Temp'] , marker='o', color='b', edgecolors='black')
plt.title('City Latitude vs. Max Temperature {}'.format(data_date))
plt.ylabel('Max Temperature (F)')
plt.xlabel('Latitude')
plt.style.use('seaborn-darkgrid')
plt.xlim(-90, 90)
plt.ylim(-100, 150)

# Save Figure
plt.savefig("analysis/Fig1.png")

# Show Figure
plt.show()
```


![png](output_12_0.png)


## Latitude vs. Humidity Plot


```python
# Create a scatter plot to show latitude and humidity
plt.scatter(Weather_df['Lat'], Weather_df['Humidity'] , marker='o', color='b', edgecolors='black')
plt.title('City Latitude vs. Humidity {}'.format(data_date))
plt.ylabel('Humidity (%)')
plt.xlabel('Latitude')
plt.style.use('seaborn-darkgrid')
plt.xlim(-90, 90)
plt.ylim(-20, 120)

# Save Figure
plt.savefig("analysis/Fig2.png")

# Show Figure
plt.show()
```


![png](output_14_0.png)


## Latitude vs. Cloudiness Plot


```python
# Create a scatter plot to show latitude and cloudiness
plt.scatter(Weather_df['Lat'], Weather_df['Cloudiness'] , marker='o', color='b', edgecolors='black')
plt.title('City Latitude vs. Cloudiness {}'.format(data_date))
plt.ylabel('Cloudiness (%)')
plt.xlabel('Latitude')
plt.style.use('seaborn-darkgrid')
plt.xlim(-90, 90)
plt.ylim(-20, 120)

# Save Figure
plt.savefig("analysis/Fig3.png")

# Show Figure
plt.show()
```


![png](output_16_0.png)


## Latitude vs. Wind Speed Plot


```python
# Create a scatter plot to show latitude and max wind speed
plt.scatter(Weather_df['Lat'], Weather_df['Wind Speed'] , marker='o', color='b', edgecolors='black')
plt.title('City Latitude vs. Wind Speed {}'.format(data_date))
plt.ylabel('Wind Speed (mph)')
plt.xlabel('Latitude')
plt.style.use('seaborn-darkgrid')
plt.xlim(-90, 90)
plt.ylim(-5, 40)

# Save Figure
plt.savefig("analysis/Fig4.png")

# Show Figure
plt.show()
```


![png](output_18_0.png)

