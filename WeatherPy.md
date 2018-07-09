

```python
%matplotlib notebook 
```


```python
# Dependencies and Setup
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np
import requests
import time
import openweathermapy as ow

# Import API key
from api_keys import api_key

# Incorporated citipy to determine city based on latitude and longitude
from citipy import citipy

# Output File (CSV)
output_data_file = "output_data/cities.csv"

# Range of latitudes and longitudes
lat_range = (-90, 90)
lng_range = (-180, 180)
```

## Generate Cities List


```python
# List for holding lat_lngs and cities
lat_lngs = []
cities = []

# Create a set of random lat and lng combinations
lats = np.random.uniform(low=-90.000, high=90.000, size=1500)
lngs = np.random.uniform(low=-180.000, high=180.000, size=1500)
lat_lngs = zip(lats, lngs)

# Identify nearest city for each lat, lng combination
for lat_lng in lat_lngs:
    city = citipy.nearest_city(lat_lng[0], lat_lng[1]).city_name
    
    # If the city is unique, then add it to a our cities list
    if city not in cities:
        cities.append(city)

# Print the city count to confirm sufficient count
len(cities)
```




    619




```python
data_def = pd.DataFrame(cities)
data = data_def.rename(columns={0:"city"})
data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ipojuca</td>
    </tr>
    <tr>
      <th>1</th>
      <td>kushima</td>
    </tr>
    <tr>
      <th>2</th>
      <td>grand gaube</td>
    </tr>
    <tr>
      <th>3</th>
      <td>north bend</td>
    </tr>
    <tr>
      <th>4</th>
      <td>norman wells</td>
    </tr>
  </tbody>
</table>
</div>




```python
data["latitude"]=""
data["longtitude"]=""
data["temperature"]=""
data["cloudiness"]=""
data["windspeed"]=""
data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>latitude</th>
      <th>longtitude</th>
      <th>temperature</th>
      <th>cloudiness</th>
      <th>windspeed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ipojuca</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>1</th>
      <td>kushima</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>2</th>
      <td>grand gaube</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>3</th>
      <td>north bend</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>4</th>
      <td>norman wells</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
</div>




```python
#testing openweather API
url = "http://api.openweathermap.org/data/2.5/weather?"
city = "cape town"
# Build query URL
query_url = url + "appid=" + api_key + "&q=" + city
test_response = requests.get(query_url).json()
# Get the temperature from the response
print(f"The weather API responded with: {test_response}.")
```

    The weather API responded with: {'coord': {'lon': 18.42, 'lat': -33.93}, 'weather': [{'id': 800, 'main': 'Clear', 'description': 'clear sky', 'icon': '01n'}], 'base': 'stations', 'main': {'temp': 287.15, 'pressure': 1022, 'humidity': 67, 'temp_min': 287.15, 'temp_max': 287.15}, 'visibility': 10000, 'wind': {'speed': 2.6, 'deg': 30}, 'clouds': {'all': 0}, 'dt': 1531166400, 'sys': {'type': 1, 'id': 6529, 'message': 0.0043, 'country': 'ZA', 'sunrise': 1531115463, 'sunset': 1531151548}, 'id': 3369157, 'name': 'Cape Town', 'cod': 200}.


## Perform API Calls


```python
counter = 0
units = "Imperial"
for index, row in data.iterrows():
    try:
        query_url = url + "appid=" + api_key + "&units=" + units + "&q=" + row["city"]
        response = requests.get(query_url).json()
        data.set_value(index, "latitude", response["coord"]["lat"])
        data.set_value(index, "longtitude", response["coord"]["lon"])
        data.set_value(index, "temperature", response["main"]["temp"])
        data.set_value(index, "humidity", response["main"]["humidity"])
        data.set_value(index, "windspeed", response["wind"]["speed"])
        data.set_value(index, "cloudiness", response["clouds"]["all"])
        
        counter += 1
        
        print( counter, response["name"])
        print(query_url)
        
    except (KeyError):
        print("Missing field/result... skipping.")
        data.drop([index])
```

    1 Ipojuca
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ipojuca


    /anaconda3/envs/PythonData/lib/python3.6/site-packages/ipykernel_launcher.py:7: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
      import sys
    /anaconda3/envs/PythonData/lib/python3.6/site-packages/ipykernel_launcher.py:8: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
      
    /anaconda3/envs/PythonData/lib/python3.6/site-packages/ipykernel_launcher.py:9: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
      if __name__ == '__main__':
    /anaconda3/envs/PythonData/lib/python3.6/site-packages/ipykernel_launcher.py:10: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
      # Remove the CWD from sys.path while we load stuff.
    /anaconda3/envs/PythonData/lib/python3.6/site-packages/ipykernel_launcher.py:11: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
      # This is added back by InteractiveShellApp.init_path()
    /anaconda3/envs/PythonData/lib/python3.6/site-packages/ipykernel_launcher.py:12: FutureWarning: set_value is deprecated and will be removed in a future release. Please use .at[] or .iat[] accessors instead
      if sys.path[0] == '':


    2 Kushima
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kushima
    3 Grand Gaube
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=grand gaube
    4 North Bend
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=north bend
    5 Norman Wells
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=norman wells
    6 Huilong
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=huilong
    7 Jakarta
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=jakarta
    Missing field/result... skipping.
    8 Bluff
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=bluff
    9 Avarua
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=avarua
    10 Constitucion
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=constitucion
    Missing field/result... skipping.
    11 Yellowknife
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=yellowknife
    12 Nanortalik
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=nanortalik
    13 Busselton
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=busselton
    14 Rikitea
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=rikitea
    15 Mokhsogollokh
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=mokhsogollokh
    16 Vaini
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=vaini
    17 Najran
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=najran
    Missing field/result... skipping.
    18 Hilo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=hilo
    19 Lompoc
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=lompoc
    20 Russell
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=russell
    21 Natal
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=natal
    22 Bredasdorp
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=bredasdorp
    23 Port Alfred
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=port alfred
    24 Cidreira
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=cidreira
    Missing field/result... skipping.
    25 Livingston
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=livingston
    26 Airai
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=airai
    27 Ushuaia
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ushuaia
    28 Ludvika
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ludvika
    29 Alamosa
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=alamosa
    30 Bethel
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=bethel
    31 Matay
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=matay
    32 Mahebourg
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=mahebourg
    33 Punta Arenas
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=punta arenas
    34 Atuona
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=atuona
    35 Salalah
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=salalah
    36 Shimoda
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=shimoda
    37 Caldwell
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=caldwell
    38 Castro
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=castro
    39 Bubaque
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=bubaque
    40 Mataura
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=mataura
    41 Nikolskoye
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=nikolskoye
    42 Aden
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=aden
    43 Tuktoyaktuk
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=tuktoyaktuk
    44 East London
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=east london
    45 Hobart
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=hobart
    46 Mar del Plata
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=mar del plata
    47 Mehamn
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=mehamn
    48 Hambantota
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=hambantota
    49 Abu Dhabi
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=abu dhabi
    50 Witu
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=witu
    51 Bereda
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=bereda
    52 Arraial do Cabo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=arraial do cabo
    53 Ife
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ife
    54 Chokurdakh
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=chokurdakh
    55 Luganville
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=luganville
    56 Cockburn Town
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=cockburn town
    57 Narsaq
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=narsaq
    58 Pangnirtung
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=pangnirtung
    Missing field/result... skipping.
    59 Buraydah
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=buraydah
    60 Cape Town
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=cape town
    61 Albany
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=albany
    62 Banda Aceh
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=banda aceh
    63 Potam
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=potam
    64 Khudumelapye
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=khudumelapye
    65 Upernavik
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=upernavik
    66 Ancud
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ancud
    67 Luzhou
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=luzhou
    68 NEDJO
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=nedjo
    Missing field/result... skipping.
    69 Saldanha
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=saldanha
    70 Presidencia Roque Saenz Pena
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=presidencia roque saenz pena
    71 Taloqan
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=taloqan
    72 Lavrentiya
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=lavrentiya
    73 Victoria
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=victoria
    74 Ambon
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ambon
    75 Aklavik
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=aklavik
    76 Dingle
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=dingle
    77 Cabo San Lucas
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=cabo san lucas
    78 Caravelas
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=caravelas
    79 Longyearbyen
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=longyearbyen
    Missing field/result... skipping.
    80 Kachiry
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kachiry
    81 Qaanaaq
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=qaanaaq
    82 Micheweni
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=micheweni
    83 Tuatapere
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=tuatapere
    84 Ponta do Sol
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ponta do sol
    85 Areia Branca
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=areia branca
    86 Hermanus
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=hermanus
    87 Moerai
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=moerai
    Missing field/result... skipping.
    88 Dikson
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=dikson
    89 Udachnyy
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=udachnyy
    90 Anage
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=anage
    91 Marsh Harbour
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=marsh harbour
    Missing field/result... skipping.
    Missing field/result... skipping.
    92 Abhar
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=abhar
    93 Vyazma
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=vyazma
    94 Port Elizabeth
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=port elizabeth
    95 Saskylakh
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=saskylakh
    96 Noyabrsk
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=noyabrsk
    97 Kudahuvadhoo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kudahuvadhoo
    98 Klyuchi
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=klyuchi
    99 Daru
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=daru
    100 Rome
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=roma
    101 Carnarvon
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=carnarvon
    102 Rodrigues Alves
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=rodrigues alves
    103 Estreito
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=estreito
    104 Pandan
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=pandan
    105 Hithadhoo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=hithadhoo
    106 Terrace
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=terrace
    107 Rorvik
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=rorvik
    108 Kruisfontein
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kruisfontein
    109 Abu Samrah
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=abu samrah
    110 Thompson
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=thompson
    111 Nepomuk
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=nepomuk
    112 Kaitangata
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kaitangata
    Missing field/result... skipping.
    113 Eydhafushi
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=eydhafushi
    114 Puerto Escondido
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=puerto escondido
    115 Elko
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=elko
    116 Zaraza
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=zaraza
    117 Puerto Ayora
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=puerto ayora
    118 Kapaa
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kapaa
    119 Pozo Colorado
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=pozo colorado
    120 Aland
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=aland
    Missing field/result... skipping.
    121 Dimasalang
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=dimasalang
    122 Vila Franca do Campo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=vila franca do campo
    Missing field/result... skipping.
    123 Portland
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=portland
    124 Praia da Vitoria
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=praia da vitoria
    125 Key West
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=key west
    126 Klaksvik
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=klaksvik
    127 Palmer
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=palmer
    128 Bartica
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=bartica
    Missing field/result... skipping.
    129 Tiksi
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=tiksi
    130 Pangoa
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=pangoa
    131 Mogapi
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=mogapi
    132 Turukhansk
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=turukhansk
    133 Butaritari
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=butaritari
    134 Esil
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=esil
    135 Tessalit
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=tessalit
    136 Khatanga
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=khatanga
    Missing field/result... skipping.
    137 Yartsevo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=yartsevo
    138 Srednekolymsk
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=srednekolymsk
    139 Maraa
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=maraa
    140 Pacific Grove
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=pacific grove
    141 Jamestown
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=jamestown
    142 Ampanihy
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ampanihy
    143 Leh
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=leh
    144 Faanui
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=faanui
    145 Fethiye
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=fethiye
    146 Los Llanos de Aridane
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=los llanos de aridane
    147 Ust-Nera
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ust-nera
    148 Kedougou
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kedougou
    149 Qeshm
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=qeshm
    150 Manggar
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=manggar
    Missing field/result... skipping.
    151 Georgetown
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=georgetown
    152 Gorontalo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=gorontalo
    153 Kupang
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kupang
    154 Jishou
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=jishou
    155 Dalnegorsk
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=dalnegorsk
    156 Hovd
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=hovd
    157 Bonavista
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=bonavista
    Missing field/result... skipping.
    158 New Norfolk
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=new norfolk
    159 Chapais
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=chapais
    160 Mayo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=mayo
    161 Ilulissat
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ilulissat
    162 Omaruru
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=omaruru
    Missing field/result... skipping.
    Missing field/result... skipping.
    163 Ukiah
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ukiah
    164 Laguna
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=laguna
    165 Outram
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=outram
    166 Severo-Kurilsk
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=severo-kurilsk
    Missing field/result... skipping.
    167 Lazaro Cardenas
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=lazaro cardenas
    168 Ballina
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ballina
    Missing field/result... skipping.
    169 Ferndale
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ferndale
    170 Kavaratti
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kavaratti
    171 Emporia
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=emporia
    172 Salinopolis
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=salinopolis
    Missing field/result... skipping.
    173 Avera
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=avera
    174 Tilichiki
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=tilichiki
    175 Balao
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=balao
    176 Vanavara
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=vanavara
    Missing field/result... skipping.
    177 Saint-Philippe
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=saint-philippe
    178 Te Anau
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=te anau
    179 Iqaluit
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=iqaluit
    180 Cherskiy
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=cherskiy
    Missing field/result... skipping.
    181 Burnie
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=burnie
    182 Santo Augusto
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=santo augusto
    183 Nome
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=nome
    184 Tucuman
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=tucuman
    185 Bratsk
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=bratsk
    186 San Patricio
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=san patricio
    187 Saint-Pierre
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=saint-pierre
    188 Huangmei
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=huangmei
    189 Ugoofaaru
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ugoofaaru
    Missing field/result... skipping.
    190 Beira
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=beira
    191 Saint-Augustin
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=saint-augustin
    192 Novodugino
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=novodugino
    193 Santander
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=santander
    194 Ust-Kuyga
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ust-kuyga
    195 Codrington
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=codrington
    196 Mogadishu
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=mogadishu
    197 Mbigou
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=mbigou
    198 Pangkalanbuun
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=pangkalanbuun
    199 Liepaja
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=liepaja
    200 Bembereke
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=bembereke
    201 Borogontsy
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=borogontsy
    202 Trairi
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=trairi
    203 Northam
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=northam
    Missing field/result... skipping.
    204 Zimapan
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=zimapan
    205 Fortuna
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=fortuna
    206 Ahipara
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ahipara
    207 Dublin
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=dublin
    208 Souillac
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=souillac
    209 Goderich
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=goderich
    210 Vila Velha
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=vila velha
    211 Berlevag
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=berlevag
    Missing field/result... skipping.
    212 Bulungu
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=bulungu
    213 Barrow
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=barrow
    214 Borova
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=borova
    215 Sitka
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=sitka
    Missing field/result... skipping.
    Missing field/result... skipping.
    216 Lianzhou
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=lianzhou
    Missing field/result... skipping.
    217 New Delhi
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=new delhi
    218 Maceo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=maceo
    219 Camacha
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=camacha
    220 Tual
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=tual
    221 Wabag
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=wabag
    222 Ribeira Grande
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ribeira grande
    223 Touros
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=touros
    224 Togur
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=togur
    225 Warmbad
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=warmbad
    Missing field/result... skipping.
    226 Assiniboia
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=assiniboia
    227 Inuvik
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=inuvik
    228 Batagay-Alyta
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=batagay-alyta
    229 Fairbanks
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=fairbanks
    Missing field/result... skipping.
    230 Roald
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=roald
    231 Batemans Bay
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=batemans bay
    232 Bilma
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=bilma
    233 Mamallapuram
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=mamallapuram
    234 Chuguyevka
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=chuguyevka
    235 Kieta
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kieta
    236 Pisco
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=pisco
    237 Isiro
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=isiro
    238 Labytnangi
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=labytnangi
    239 Vestmannaeyjar
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=vestmannaeyjar
    240 Tasiilaq
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=tasiilaq
    241 Muravlenko
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=muravlenko
    242 Luderitz
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=luderitz
    Missing field/result... skipping.
    243 Pouebo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=pouebo
    244 Hofn
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=hofn
    245 Yulara
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=yulara
    246 Sassandra
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=sassandra
    247 Lorengau
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=lorengau
    248 Biloela
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=biloela
    249 Bentley
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=bentley
    250 Bismarck
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=bismarck
    251 Dunedin
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=dunedin
    Missing field/result... skipping.
    252 Acapulco
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=acapulco
    253 Udarnyy
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=udarnyy
    254 Takoradi
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=takoradi
    255 Along
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=along
    256 High Rock
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=high rock
    257 Oktyabrskoye
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=oktyabrskoye
    Missing field/result... skipping.
    258 Mayumba
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=mayumba
    259 Port Blair
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=port blair
    260 Tezu
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=tezu
    261 Pokrovskoye
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=pokrovskoye
    262 Saint George
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=saint george
    263 Walvis Bay
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=walvis bay
    264 Hami
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=hami
    265 Masvingo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=masvingo
    266 Bambous Virieux
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=bambous virieux
    Missing field/result... skipping.
    267 Mocajuba
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=mocajuba
    268 Lata
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=lata
    269 Mednogorsk
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=mednogorsk
    270 Saint-Francois
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=saint-francois
    271 Namatanai
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=namatanai
    272 Bathsheba
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=bathsheba
    273 Paamiut
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=paamiut
    274 Kavieng
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kavieng
    275 Omboue
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=omboue
    276 Stepnogorsk
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=stepnogorsk
    277 Richards Bay
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=richards bay
    278 Kutiyana
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kutiyana
    279 Dunnville
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=dunnville
    280 Torbay
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=torbay
    Missing field/result... skipping.
    281 Moose Factory
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=moose factory
    282 San Quintin
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=san quintin
    283 Rio Grande
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=rio grande
    284 Vanino
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=vanino
    285 Sambava
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=sambava
    286 Opuwo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=opuwo
    287 Marawi
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=marawi
    288 Baghmara
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=baghmara
    289 Rio Gallegos
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=rio gallegos
    290 Coahuayana
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=coahuayana
    291 Coronel Oviedo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=coronel oviedo
    Missing field/result... skipping.
    292 Port Macquarie
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=port macquarie
    293 Plover
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=plover
    Missing field/result... skipping.
    294 Pravia
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=pravia
    295 Gushikawa
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=gushikawa
    296 Burns Lake
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=burns lake
    Missing field/result... skipping.
    297 Amvrosiyivka
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=amvrosiyivka
    Missing field/result... skipping.
    298 Mildura
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=mildura
    299 Plerin
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=plerin
    300 Sheregesh
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=sheregesh
    301 Clonakilty
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=clonakilty
    302 Blyznyuky
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=blyznyuky
    303 Neiafu
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=neiafu
    304 Caraballeda
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=caraballeda
    Missing field/result... skipping.
    305 Jutiapa
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=jutiapa
    306 Kozhva
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kozhva
    307 Waddan
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=waddan
    308 Corumba
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=corumba
    309 Egvekinot
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=egvekinot
    310 Karratha
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=karratha
    311 Ternate
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ternate
    Missing field/result... skipping.
    312 Viedma
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=viedma
    313 Saint-Raphael
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=saint-raphael
    314 Proletariy
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=proletariy
    315 Port Hawkesbury
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=port hawkesbury
    316 Barauli
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=barauli
    317 Brodokalmak
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=brodokalmak
    318 Douglas
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=douglas
    319 Paulo Afonso
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=paulo afonso
    320 Huangpi
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=huangpi
    321 Fernley
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=fernley
    322 Pierre
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=pierre
    323 Lakes Entrance
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=lakes entrance
    324 Sao Joao da Barra
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=sao joao da barra
    325 Meulaboh
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=meulaboh
    326 Kamalapuram
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kamalapuram
    Missing field/result... skipping.
    Missing field/result... skipping.
    327 Tete
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=tete
    328 Tsaratanana
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=tsaratanana
    329 Atyrau
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=atyrau
    330 Saint Anthony
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=saint anthony
    331 Hirara
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=hirara
    332 Alice Springs
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=alice springs
    333 Esperance
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=esperance
    334 Kodiak
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kodiak
    335 Ippy
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ippy
    336 Lebu
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=lebu
    337 Jiguani
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=jiguani
    338 Mount Gambier
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=mount gambier
    339 Kropotkin
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kropotkin
    340 Waitati
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=waitati
    Missing field/result... skipping.
    341 Zemio
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=zemio
    342 Shenjiamen
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=shenjiamen
    343 Praya
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=praya
    344 Naze
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=naze
    345 Aksu
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=aksu
    346 Kysyl-Syr
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kysyl-syr
    347 Matara
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=matara
    Missing field/result... skipping.
    Missing field/result... skipping.
    348 Leningradskiy
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=leningradskiy
    349 Kahului
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kahului
    350 Emba
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=emba
    351 San Policarpo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=san policarpo
    352 Katsuura
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=katsuura
    353 Copiapo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=copiapo
    354 Sao Filipe
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=sao filipe
    355 La Ronge
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=la ronge
    356 Krivosheino
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=krivosheino
    357 San Miguel
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=san miguel
    358 Amahai
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=amahai
    359 Ingham
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ingham
    360 Faya
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=faya
    361 Larsnes
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=larsnes
    362 Gimli
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=gimli
    363 Carutapera
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=carutapera
    364 Pevek
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=pevek
    365 Lodja
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=lodja
    366 San Anselmo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=san anselmo
    367 General Roca
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=general roca
    368 Chuy
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=chuy
    Missing field/result... skipping.
    369 Novobureyskiy
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=novobureyskiy
    370 Novouzensk
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=novouzensk
    Missing field/result... skipping.
    371 Cabedelo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=cabedelo
    372 Alofi
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=alofi
    373 Liverpool
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=liverpool
    Missing field/result... skipping.
    374 Garissa
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=garissa
    375 Durban
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=durban
    376 San Cristobal
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=san cristobal
    377 El Astillero
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=el astillero
    378 Dutlwe
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=dutlwe
    379 Damietta
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=damietta
    380 Thinadhoo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=thinadhoo
    381 Bowen
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=bowen
    382 Vao
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=vao
    383 Bonthe
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=bonthe
    384 Port Huron
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=port huron
    385 Bend
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=bend
    Missing field/result... skipping.
    386 Marystown
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=marystown
    387 Ouesso
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ouesso
    388 Provideniya
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=provideniya
    389 Kendari
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kendari
    390 Russkaya Polyana
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=russkaya polyana
    391 Sangar
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=sangar
    392 Adrar
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=adrar
    393 Coihaique
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=coihaique
    394 Imbituba
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=imbituba
    395 Liaozhong
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=liaozhong
    Missing field/result... skipping.
    396 Margate
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=margate
    397 San Borja
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=san borja
    398 Oussouye
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=oussouye
    399 Lasa
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=lasa
    400 Zaliztsi
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=zaliztsi
    401 Nicoya
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=nicoya
    Missing field/result... skipping.
    402 Tura
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=tura
    403 Kamaishi
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kamaishi
    404 Plettenberg Bay
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=plettenberg bay
    405 Los Andes
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=los andes
    406 Jolo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=jolo
    407 Pindobacu
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=pindobacu
    Missing field/result... skipping.
    408 Kiruna
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kiruna
    409 Ouidah
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ouidah
    410 Chesterville
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=chesterville
    411 Puerto Ayacucho
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=puerto ayacucho
    412 Poum
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=poum
    413 Panguna
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=panguna
    414 Druskininkai
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=druskininkai
    415 Taltal
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=taltal
    416 Calabozo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=calabozo
    417 Haines Junction
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=haines junction
    418 Eureka
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=eureka
    419 Buala
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=buala
    Missing field/result... skipping.
    Missing field/result... skipping.
    420 San Jose
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=san jose
    Missing field/result... skipping.
    421 Ulagan
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ulagan
    422 Mandan
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=mandan
    423 Dzaoudzi
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=dzaoudzi
    Missing field/result... skipping.
    424 Wanning
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=wanning
    425 Sobolevo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=sobolevo
    426 Wuchi
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=wuchi
    427 Beringovskiy
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=beringovskiy
    Missing field/result... skipping.
    428 Braganca
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=braganca
    429 Oskemen
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=oskemen
    430 Vryburg
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=vryburg
    431 Ust-Tsilma
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ust-tsilma
    432 Novyy Urengoy
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=novyy urengoy
    433 Terney
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=terney
    434 Raudeberg
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=raudeberg
    435 Hasaki
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=hasaki
    436 Waipawa
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=waipawa
    437 Coos Bay
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=coos bay
    438 Sidhi
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=sidhi
    439 Nong Khae
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=nong khae
    440 Seredka
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=seredka
    441 Masallatah
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=masallatah
    442 Wucheng
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=wucheng
    443 Maningrida
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=maningrida
    444 Oxelosund
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=oxelosund
    445 Isangel
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=isangel
    446 Benguela
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=benguela
    447 Sao Francisco de Paula
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=sao francisco de paula
    448 Puerto Leguizamo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=puerto leguizamo
    449 Merauke
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=merauke
    450 Camopi
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=camopi
    451 Phalodi
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=phalodi
    452 Westport
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=westport
    453 Saquarema
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=saquarema
    454 Hualmay
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=hualmay
    455 Nizhniy Tsasuchey
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=nizhniy tsasuchey
    456 Xining
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=xining
    457 Chimoio
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=chimoio
    458 Baker City
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=baker city
    459 Formosa
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=formosa
    460 Clyde River
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=clyde river
    461 Penzance
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=penzance
    Missing field/result... skipping.
    462 Nemuro
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=nemuro
    Missing field/result... skipping.
    463 Guerrero Negro
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=guerrero negro
    464 Yining
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=yining
    465 Lensk
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=lensk
    466 Ibra
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ibra
    467 Geraldton
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=geraldton
    468 Vila do Maio
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=vila do maio
    Missing field/result... skipping.
    469 Chernyshevskiy
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=chernyshevskiy
    470 Kibala
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kibala
    Missing field/result... skipping.
    471 Broome
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=broome
    472 Carauari
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=carauari
    473 Washington DC.
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=washington
    474 Miraflores
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=miraflores
    475 Sahuaripa
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=sahuaripa
    476 Cabrero
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=cabrero
    477 Fort-Shevchenko
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=fort-shevchenko
    478 Pakpattan
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=pakpattan
    479 Xifeng
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=xifeng
    480 Emet
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=emet
    481 Sabang
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=sabang
    482 Nenjiang
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=nenjiang
    483 Namibe
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=namibe
    484 Kaeo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kaeo
    485 Arona
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=arona
    486 Sao Jose da Coroa Grande
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=sao jose da coroa grande
    Missing field/result... skipping.
    487 Metro
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=metro
    488 San Andres
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=san andres
    489 Zhongshu
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=zhongshu
    490 Wareham
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=wareham
    491 Alyth
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=alyth
    492 Grindavik
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=grindavik
    Missing field/result... skipping.
    493 Marzuq
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=marzuq
    494 Vardo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=vardo
    495 Opelika
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=opelika
    496 Pingliang
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=pingliang
    497 Robertson
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=robertson
    498 Wyndham
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=wyndham
    499 Yar-Sale
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=yar-sale
    Missing field/result... skipping.
    500 Jining
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=jining
    501 Gat
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=gat
    Missing field/result... skipping.
    502 Lady Frere
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=lady frere
    503 Ust-Karsk
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ust-karsk
    504 Navoi
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=navoi
    505 Lodeynoye Pole
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=lodeynoye pole
    506 Coquimbo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=coquimbo
    507 Nan
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=nan
    Missing field/result... skipping.
    508 Henties Bay
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=henties bay
    509 Khandyga
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=khandyga
    Missing field/result... skipping.
    510 Pointe Michel
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=pointe michel
    511 Abashiri
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=abashiri
    512 Morshansk
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=morshansk
    513 Kalmunai
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kalmunai
    514 Mount Isa
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=mount isa
    515 Chakia
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=chakia
    516 Dukat
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=dukat
    517 Hongjiang
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=hongjiang
    518 Ussel
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=ussel
    519 Novyy Urgal
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=novyy urgal
    520 Worland
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=worland
    521 Carepa
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=carepa
    522 Santa Vitoria do Palmar
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=santa vitoria do palmar
    Missing field/result... skipping.
    523 Kuching
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kuching
    524 Solnechnyy
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=solnechnyy
    525 Kidal
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kidal
    526 Turtas
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=turtas
    527 Kresttsy
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kresttsy
    528 Emerald
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=emerald
    529 Nouadhibou
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=nouadhibou
    530 Kutum
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=kutum
    531 Hermiston
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=hermiston
    532 Chazuta
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=chazuta
    533 Comodoro Rivadavia
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=comodoro rivadavia
    534 Cordoba
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=cordoba
    535 Mutis
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=mutis
    536 Atascocita
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=atascocita
    537 Fayaoue
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=fayaoue
    538 Porto Novo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=porto novo
    539 Andra
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=andra
    540 Manzanillo
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=manzanillo
    541 Verkh-Usugli
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=verkh-usugli
    542 Isabela
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=isabela
    Missing field/result... skipping.
    543 Nago
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=nago
    Missing field/result... skipping.
    544 Moba
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=moba
    545 Prudy
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=prudy
    546 Fiumicino
    http://api.openweathermap.org/data/2.5/weather?appid=6a1c2868d6932def9cd19f54d5d9e2d4&units=Imperial&q=fiumicino



```python
#take 500 sample cities
raw_data = data.dropna()
weather = raw_data.sample(500).reset_index()
weather.head(25)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>city</th>
      <th>latitude</th>
      <th>longtitude</th>
      <th>temperature</th>
      <th>cloudiness</th>
      <th>windspeed</th>
      <th>humidity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>160</td>
      <td>los llanos de aridane</td>
      <td>28.66</td>
      <td>-17.92</td>
      <td>71.6</td>
      <td>75</td>
      <td>13.87</td>
      <td>83.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>402</td>
      <td>la ronge</td>
      <td>55.1</td>
      <td>-105.3</td>
      <td>77</td>
      <td>75</td>
      <td>6.93</td>
      <td>38.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>442</td>
      <td>sangar</td>
      <td>63.92</td>
      <td>127.47</td>
      <td>71.41</td>
      <td>68</td>
      <td>4.16</td>
      <td>62.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>60</td>
      <td>narsaq</td>
      <td>60.91</td>
      <td>-46.05</td>
      <td>51.8</td>
      <td>36</td>
      <td>8.05</td>
      <td>62.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>586</td>
      <td>hongjiang</td>
      <td>27.21</td>
      <td>109.83</td>
      <td>72.04</td>
      <td>32</td>
      <td>2.48</td>
      <td>97.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>234</td>
      <td>goderich</td>
      <td>43.74</td>
      <td>-81.71</td>
      <td>83.92</td>
      <td>0</td>
      <td>14.12</td>
      <td>44.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>338</td>
      <td>mildura</td>
      <td>-34.18</td>
      <td>142.16</td>
      <td>47.74</td>
      <td>76</td>
      <td>4.5</td>
      <td>95.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>288</td>
      <td>along</td>
      <td>28.17</td>
      <td>94.8</td>
      <td>71.59</td>
      <td>88</td>
      <td>0.92</td>
      <td>98.0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>485</td>
      <td>beringovskiy</td>
      <td>63.05</td>
      <td>179.32</td>
      <td>47.92</td>
      <td>8</td>
      <td>8.75</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>566</td>
      <td>jining</td>
      <td>35.41</td>
      <td>116.58</td>
      <td>71.5</td>
      <td>92</td>
      <td>5.28</td>
      <td>98.0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>156</td>
      <td>ampanihy</td>
      <td>-24.69</td>
      <td>44.75</td>
      <td>61.42</td>
      <td>0</td>
      <td>11.54</td>
      <td>82.0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>74</td>
      <td>saldanha</td>
      <td>41.42</td>
      <td>-6.55</td>
      <td>79.24</td>
      <td>48</td>
      <td>6.51</td>
      <td>34.0</td>
    </tr>
    <tr>
      <th>12</th>
      <td>18</td>
      <td>najran</td>
      <td>17.54</td>
      <td>44.22</td>
      <td>89.6</td>
      <td>40</td>
      <td>4.7</td>
      <td>18.0</td>
    </tr>
    <tr>
      <th>13</th>
      <td>317</td>
      <td>moose factory</td>
      <td>51.26</td>
      <td>-80.61</td>
      <td>71.6</td>
      <td>75</td>
      <td>12.75</td>
      <td>49.0</td>
    </tr>
    <tr>
      <th>14</th>
      <td>201</td>
      <td>iqaluit</td>
      <td>63.75</td>
      <td>-68.52</td>
      <td>46.4</td>
      <td>90</td>
      <td>8.05</td>
      <td>61.0</td>
    </tr>
    <tr>
      <th>15</th>
      <td>266</td>
      <td>kieta</td>
      <td>-6.22</td>
      <td>155.63</td>
      <td>76.9</td>
      <td>48</td>
      <td>3.15</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>16</th>
      <td>52</td>
      <td>abu dhabi</td>
      <td>24.47</td>
      <td>54.37</td>
      <td>93.2</td>
      <td>0</td>
      <td>4.7</td>
      <td>75.0</td>
    </tr>
    <tr>
      <th>17</th>
      <td>192</td>
      <td>salinopolis</td>
      <td>-0.61</td>
      <td>-47.36</td>
      <td>83.47</td>
      <td>36</td>
      <td>12.21</td>
      <td>92.0</td>
    </tr>
    <tr>
      <th>18</th>
      <td>407</td>
      <td>faya</td>
      <td>18.39</td>
      <td>42.45</td>
      <td>77.88</td>
      <td>0</td>
      <td>4.94</td>
      <td>26.0</td>
    </tr>
    <tr>
      <th>19</th>
      <td>511</td>
      <td>westport</td>
      <td>53.8</td>
      <td>-9.52</td>
      <td>59</td>
      <td>20</td>
      <td>11.41</td>
      <td>67.0</td>
    </tr>
    <tr>
      <th>20</th>
      <td>529</td>
      <td>vila do maio</td>
      <td>15.13</td>
      <td>-23.22</td>
      <td>78.07</td>
      <td>48</td>
      <td>3.04</td>
      <td>92.0</td>
    </tr>
    <tr>
      <th>21</th>
      <td>66</td>
      <td>banda aceh</td>
      <td>5.56</td>
      <td>95.32</td>
      <td>76.27</td>
      <td>64</td>
      <td>5.73</td>
      <td>92.0</td>
    </tr>
    <tr>
      <th>22</th>
      <td>61</td>
      <td>pangnirtung</td>
      <td>66.15</td>
      <td>-65.72</td>
      <td>46.4</td>
      <td>75</td>
      <td>9.17</td>
      <td>70.0</td>
    </tr>
    <tr>
      <th>23</th>
      <td>590</td>
      <td>carepa</td>
      <td>7.76</td>
      <td>-76.65</td>
      <td>80.6</td>
      <td>75</td>
      <td>8.05</td>
      <td>88.0</td>
    </tr>
    <tr>
      <th>24</th>
      <td>509</td>
      <td>camopi</td>
      <td>3.17</td>
      <td>-52.33</td>
      <td>74.56</td>
      <td>92</td>
      <td>2.37</td>
      <td>87.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
#print(f"The weather API responded with: {test_response}.")
print(weather["temperature"].mean())
print(weather["windspeed"].mean())
print(weather["humidity"].mean())
```

    67.88196000000005
    8.113919999999995
    73.498


## Temperature (F) vs. Latitude


```python
plt.scatter(weather["latitude"], weather["temperature"], marker="o", color = 'blue')

# # Incorporate the other graph properties
plt.title("Temperature (F) vs. Latitude ")
plt.ylabel("Temperature (F)")
plt.xlabel("Latitude")
plt.grid()


plt.show()
```


    <IPython.core.display.Javascript object>



<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAoAAAAHgCAYAAAA10dzkAAAAAXNSR0IArs4c6QAAQABJREFUeAHsfQn8XcP5/liCxFaxLyGprVqq2qq1RImltqqlaFVQtErFTilJCUFr32rpn4pYQ0opCYKqpdTS1k8RhGqsFbGFBOc/zznee+fOnTlnzr3n3u+99zzv53PvOWfOOzPvPDPnzHvemXlnjkiTIhEBIkAEiAARIAJEgAiUBoE5S1NSFpQIEAEiQASIABEgAkQgRoAKIBsCESACRIAIEAEiQARKhgAVwJJVOItLBIgAESACRIAIEAEqgGwDRIAIEAEiQASIABEoGQJUAEtW4SwuESACRIAIEAEiQASoALINEAEiQASIABEgAkSgZAhQASxZhbO4RIAIEAEiQASIABGgAsg2QASIABEgAkSACBCBkiFABbBkFc7iEgEiQASIABEgAkSACiDbABEgAkSACBABIkAESoYAFcCSVTiLSwSIABEgAkSACBABKoBsA0SACBABIkAEiAARKBkCVABLVuEsLhEgAkSACBABIkAEqACyDRABIkAEiAARIAJEoGQIUAEsWYWzuESACBABIkAEiAARoALINkAEiAARIAJEgAgQgZIhQAWwZBXO4hIBIkAEiAARIAJEgAog2wARIAJEgAgQASJABEqGABXAklU4i0sEiAARIAJEgAgQASqAbANEgAgQASJABIgAESgZAlQAS1bhLC4RIAJEgAgQASJABKgAsg0QASJABIgAESACRKBkCFABLFmFs7hEgAgQASJABIgAEaACyDZABIgAESACRIAIEIGSIUAFsGQVzuISASJABIgAESACRIAKINsAESACRIAIEAEiQARKhgAVwJJVOItLBIgAESACRIAIEAEqgGwDRIAIEAEiQASIABEoGQJUAEtW4SwuESACRIAIEAEiQASoALINEAEiQASIABEgAkSgZAhQASxZhbO4RIAIEAEiQASIABGgAsg2QASIABEgAkSACBCBkiFABbBkFc7iEgEiQASIABEgAkSACiDbABEgAkSACBABIkAESoYAFcCSVTiLSwSIABEgAkSACBABKoBsA0SACBABIkAEiAARKBkCVABLVuEsbjEIzDHHHCrkd8899xSTYZen8oc//EGdd955HV2KSZMmqQEDBqhp06ZV5Fx33XW99TxlypSY79Zbb1ULL7yweuONNyrxOu3koosuisvxr3/9qxDRfv3rX6s//elPdWndfvvtcT4PPfRQ5d7NN9+sTjrppMp1kSdHH320mm+++YpMkmkRgdIgMHdpSsqCEoECEXjwwQdrUjvxxBPV5MmT1d13310T/uUvf7nmuqwXUABfeeUVdeCBB3YkBJ9++qk69NBD1QEHHKCWWWaZGhm/9KUvqf/3//5fTRguBg0aFIdtvfXW6itf+Yr61a9+pX73u9/V8fViABTAn/zkJ2qbbbapKd56662n8GysvvrqlXAogGPHjlXHHXdcJYwnRIAI9D0CVAD7vg4oQRciAMuQSYsvvriac845lR1u8vTS+cyZM1X//v37vEgffvhhbLVrVhAoKU899ZS65ZZb6pKaf/75M+v15z//udprr70UFKMll1yyLo2yBMASWpZnoCx1ynL2LgIcAu7dumXJOgiBd955Rx1yyCFq8ODBap555omtR4cffriCIiX00UcfxcNnCIclaeWVV46VrHXWWUf9/e9/V5999pk6+eST1QorrKAWXHBBtfnmm6upU6dK9PiIzveb3/xmbIlce+214+Gx5ZZbLlZMEN+kjz/+WI0cOVKtuuqqat55540Vl3333Vf973//M9nUUkstpXbaaSd1zTXXqDXXXDPmPfXUU2Oes846S2244YYKCvACCywQ3z/jjDPUJ598UkkDMt11113qmWeeqQynyrCda8gQEf/973/HvMhTaNddd1WLLbaYevzxx9Wmm24a5/fd735Xbqs///nPaujQoTE2GMrdaKON1H333Ve5n3Zy4YUXxuVA/TRCO+ywQ1yvl112WWr0MWPGqLnmmku9/PLLdXwHH3xwXN8zZsyI7z3yyCNqq622irFF/Sy77LJq2223Va+99lpd3CICPvjgg7iNfvWrX1ULLbSQWnTRRdUGG2ygMMQtJG0UFlO0UZkGseWWW8Ysdn2izoAJ2prw4ogyuOoYiUgewMqkCRMmqDXWWCNufyuuuKI6++yzzduVc7Rz3EM50M4GDhyofvCDH6iXXnqpwsMTIkAElKIFkK2ACLQYgffeey9WLt566y31y1/+Mh4ufPLJJ2PlC1YnKC4m3XDDDbGSd/rppyt0tEceeaSCooNO7NVXX1VQVqCkYchyl112UX/729/M6LFysccee8RDbl/84hcVrFsnnHCCevfdd9VvfvOb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xQJnrcADM1tmzqzNx3UFvk77bm0FHq6yd0sY8ajWVDNYIG7ZqSsVQLhsmTJlSqXuXnzxxdjX38CBA9Xyyy+v3taTWl5++WU1bdq0mOeZZ56Jj7D84bfwwgurffbZRx122GFq0UUXVYh3+OGHqzXWWKOyKriSOE+IABEgAl2OAFb7hlAoX0ha5CECRKCzEejKVcCPPvqoWmutteIf4MWwLK6PP/74GO2bb745vt768zGPXXfdNb6+6KKLKrVx5plnqu9973uxBRA+AwcMGKBu0bOk56I/hApGPCECRKA3EICrl+WWU3qOs7s8CNezWWKXMG4OhhIBItBrCHSlBXDo0KF6JZteyuah4cOHK/zSaL755lPnnntu/Evj4z0iQASIQLcjgO/as89WaqedEiXQfH2KUnjWWfQH2O31TPmJQB4EutICmKeA5CUCRIAIEAGlvv99pW64Qalll61FA5ZBhOM+iQgQgfIg0JUWwPJUD0tKBIgAESgOASh522/PnUCKQ5QpEYHuRYAKYPfWHSUnAkSACORGAMPBehYNiQgQgZIjwCHgkjcAFp8IEAEiQASIABEoHwJUAMtX5ywxESACRIAIEAEiUHIEqACWvAGw+ESACBABIkAEiED5EKACWL46Z4mJABEgAkSACBCBkiNABbDkDYDFJwJEgAgQASJABMqHABXA8tU5S0wEiAARIAJEgAiUHAEqgCVvACw+ESACRIAIEAEiUD4EqACWr85ZYiJABIgAESACRKDkCFABLHkDYPGJABEgAkSACBCB8iFABbB8dc4SEwEiQASIABEgAiVHgApgyRsAi08EiAARIAJEgAiUDwEqgOWrc5aYCBABIkAEiAARKDkCVABL3gBYfCJABIgAESACRKB8CFABLF+ds8REgAgQASJABIhAyRGgAljyBsDiEwEiQASIABEgAuVDgApg+eqcJSYCRIAIEAEiQARKjgAVwJI3ABafCBABIkAEiAARKB8CVADLV+csMREgAkSACBABIlByBKgAlrwBsPhEgAgQASJABIhA+RCYu3xFZomJABEgAuVA4NNPlfrLX5R69VWlll5aqW9/W6m55ipH2VlKIkAE0hGgApiOD+8SASJABLoSgRtvVOrgg5V65ZWq+Mstp9TZZyv1/e9Xw3hGBIhAORHgEHA5652lJgJEoIcRgPK30061yh+K+9//JuG4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QAMwK3K8jggQASJABIgAESACbYoADcCcb9zdd98t6623nvTp00eWW245+elPfxoo4f/+7/9ku+22k379+nnnjzzySJk/f36ApmoH8+bNk7XWWstbLeW5554LVO/555+XjTfe2MNrxRVXlP/4j/+Qqi1O89Zbb8n48eNl+PDhXj1XWWUVOfXUU7vd907Awrz5l1xyiYfJ4osvLuuss478+c9/Nk9Xcv/ss8+Wdddd11s9aPnll5cddthBXnnllUBd8b4cccQRnn6Anth+++3l3XffDdBU9QD4YFWlo446ql7FTsPj73//u+y5554yYMAA6du3r6c7n3nmmToe0I+YvAMrWOA7gxknXnzxxfr5Ku189dVXcsopp9R158orr+x9I7AKl06dhIeuc25bBR5TTgj84Q9/qC2zzDK1Sy+9tKaUeu3ll1+u/f73v69zVw9zbY011qhtsskmtWeffbY2ZcqUmnqJaz/72c/qNFXcUUZubauttsKSg7W//e1v9SrOmjWrNnDgwNquu+5aU8ZPbfLkyTW1rF7t/PPPr9NUYefee++t7bvvvrX777+/9sYbb9TuuOOOmvr414455ph69ToFC13hW265pdarV6/alVdeWXvppZdqP//5z2vK2Km9/fbbmqSS2y222KJ29dVX11544YWaagzVttlmm9pKK61U+/zzz+v1PeSQQ2qqMeTpB+gJ6Ivvf//7NeiPKqe//vWvtWHDhtXWXHNN73nQde0kPD7++OPat7/9bU9fPPnkk7UZM2bUHnzwwdrrr7+u4aidc845np6EvoTe3GWXXWorrLBCTS2PVqepys4ZZ5xRU4Zw7U9/+pOHBb6nSyyxRO2iiy6qV7GT8KhXOqcdeFuYckBgwYIFntL+r//6r0hu99xzT00tH1dTLbw6zc0331xbbLHFajAAqphQ5+985zs11ULtZgAqD1BNLcVT+/LLL+tVVx4AzyhWLbx6XhV3Jk6cWFMewXrVOg2LH/zgBzV82M2E5+TEE080syq//9FHH3nvxaOPPurV9dNPP/UMYxjIOkFfQG/cd999OqtyW7Uma23EiBGe0at6BOoGYKfhccIJJ9Q23HDDyPsLvTho0CDPCNRE0J/Qo5dddpnOqswWDaT9998/UB/Vq1ZTHlIvr9PwCACRwwG7gHPypaqWusB1j/WB1157bVEtMlFer4Br/okn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QEaOHCn9+vWToUOHymGHHSaff/65R/LII4/IfvvtJ7NmzRJ4+PBTa09758wuYOwj/eQnP/Fo9HGY5xFdu+ji1emLL76QvffeW5ZYYglZYYUV5Fe/+pU+Vd+qNcNFLWYvK664oifneuutJ5CNiQgQASJQJAI0AItEm2URASLQVAQWWWQR+fWvfy0vvPCCXHvttfLf//3fnrGFQtdff30vzq9///7y/vvve79jjz22mzzoDka6+uqrPRp93I0wJOO4446Thx9+WG677TZ54IEHPMPumWeeCVDCCP3LX/4it9xyi0yfPl122mkn2XLLLeW1114L0PGACBABItBMBNgF3Ex0yZsIEIFCETAHWwwfPlxOP/10OfTQQ+WSSy6R3r17y1JLLeV59QYNGhQpl+4ORqxgHJ3NAJ7Gq666Sq677joZN26cdxpG6JAhQ+qkb7zxhtx8883y7rvvyuDBg718GKH33XefZ3CeddZZdVruEAEiQASaiQANwGaiS95EgAgUigC8bzCiXnrpJZk9e7Z89dVX8uWXXwq6ZtEt3MwE4w7du6NHj64Xs+yyy8rqq69eP3722WelVqvJaqutVs/Dzrx582TAgAGBPB4QASJABJqJAA3AZqJL3kSACBSGwNtvvy1bb721HHLIIZ7nD8bX1KlTZfz48bJgwYKG5UD3Mow3M5l87XMmnd5fuHChLLroooJuYWzNhLhBJiJABIhAUQjQACwKaZZDBIhAUxF4+umnPY8fBl7AWEO69dZbA2WiG/jrr78O5IUd9OrVqxsduoYRW2im5557TkCLtOqqq3r706ZNk5VWWsnL++STT+TVV1+VjTfe2Dtee+21Pb4fffSR/OhHP/Ly+EcEiAARaAUCHATSCtRZJhEgAg0hgJG8ML7MHww0dPlOmjRJ3nzzTbn++uvlsssuC5SDEb2I1XvooYdk5syZMmfOnMB5fQA60HzwwQcCIw5p0003FRiZiPHDgI1TTz01YBDCgwdvIwaC4FoYixg5rI1R8EDX7x577OGNFP7jH/8oM2bMEAwyOffcc+Wee+4BCRMRIAJEoBAEaAAWAjMLIQJEIE8EMG0KvGnm77e//a1gGhgYU2ussYbceOONcvbZZweKxUhgdBHvsssuAoNx4sSJgfP6AF7EKVOmeFPJoAykLbbYQn75y196o4rXXXdd+eyzzzxDTl+D7XnnnScbbbSRbL/99rLZZpvJhhtuKOuss45J4g32wFQxxxxzjBcfCNonn3zSKytAyAMiQASIQBMR6KHiVoJBLU0sjKyJABEgAkSACBABIkAEWo8APYCtvweUgAgQASJABIgAESAChSJAA7BQuFkYESACRIAIEAEiQARajwANwNbfA0pABIgAESACRIAIEIFCEaABWCjcLIwIEAEiQASIABEgAq1HgAZg6+8BJSACRIAIEAEiQASIQKEI0AAsFG4WRgSIABEgAkSACBCB1iNAA7D194ASEAEiQASIABEgAkSgUARoABYKNwsjAkSACBABIkAEiEDrEaAB2Pp7QAmIABEgAkSACBABIlAoAjQAC4WbhREBIkAEiAARIAJEoPUI0ABs/T2gBESACBABIkAEiAARKBQBGoCFws3CiAARIAJEgAgQASLQegT+P+T8nPdRDC7jAAAAAElFTkSuQmCC" width="640">


## Humidity (%) vs. Latitude


```python
plt.scatter(weather["latitude"], weather["humidity"], marker="o", color = 'blue')

# # Incorporate the other graph properties
plt.title("Humidity (%) vs. Latitude ")
plt.ylabel("Humidity (%)")
plt.xlabel("Latitude")
plt.grid()


plt.show()
```


    <IPython.core.display.Javascript object>



<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAoAAAAHgCAYAAAA10dzkAAAAAXNSR0IArs4c6QAAQABJREFUeAHsfQfYZjWVfxgcYEaKSxcYHAR0LdiRKgxS1qWoa0FAECyA4ArSpfgwsKJ0XEGW4ioKzCj7HwQRFJGmSNNVFsRCEXhUliYuRQYG5P3nd6/n+86bL+XklreePM/33dzk5JRfcnPzJucmi3VsMBoUAUVAEVAEFAFFQBFQBMYGgWljY6kaqggoAoqAIqAIKAKKgCJQIKADQG0IioAioAgoAoqAIqAIjBkCOgAcswpXcxUBRUARUAQUAUVAEdABoLYBRUARUAQUAUVAEVAExgwBHQCOWYWruYqAIqAIKAKKgCKgCOgAUNuAIqAIKAKKgCKgCCgCY4aADgDHrMLVXEVAEVAEFAFFQBFQBHQAqG1AEVAEFAFFQBFQBBSBMUNAB4BjVuFqriKgCCgCioAioAgoAjoA1DagCCgCioAioAgoAorAmCGgA8Axq3A1VxFQBBQBRUARUAQUAR0AahtQBBQBRUARUAQUAUVgzBDQAeCYVbiaqwgoAoqAIqAIKAKKgA4AtQ0oAoqAIqAIKAKKgCIwZgjoAHDMKlzNVQQUAUVAEVAEFAFFQAeA2gYUAUVAEVAEFAFFQBEYMwR0ADhmFa7mKgKKgCKgCCgCioAioANAbQOKgCKgCCgCioAioAiMGQI6AByzCldzFQFFQBFQBBQBRUAR0AGgtgFFQBFQBBQBRUARUATGDAEdAI5Zhau5ioAioAgoAoqAIqAI6ABQ24AioAgoAoqAIqAIKAJjhoAOAMeswtVcRUARUAQUAUVAEVAEdACobUARUAQUAUVAEVAEFIExQ0AHgGNW4WquIqAIKAKKgCKgCCgCOgDUNqAIKAKKgCKgCCgCisCYIaADwDGrcDVXEVAEFAFFQBFQBBQBHQBqG1AEFAFFQBFQBBQBRWDMENAB4JhVuJqrCCgCioAioAgoAoqADgC1DSgCioAioAgoAoqAIjBmCOgAcMwqXM1VBBQBRUARUAQUAUVAB4DaBhQBRUARUAQUAUVAERgzBHQAOGYVruYqAoqAIqAIKAKKgCKgA0BtA4qAIqAIKAKKgCKgCIwZAjoAHLMKV3MVAUVAEVAEFAFFQBHQAaC2AUVAEVAEFAFFQBFQBMYMAR0AjlmFq7n9ReC8884ziy22mPn5z3/uVWT77bc3s2fP9ua1nXj//fcXukHHVJg7d25By+mg9x577DGR9OCDDxrQ3XbbbRNpTUaOPfZY89rXvta8+OKLBdtOp2OOPvpos/rqq5uVV17Z7Lfffua5557rEvnEE0+Y1VZbzXzta1/rSsfN1VdfbZZeemnzpz/9aUreoCSk2k+unl/4whfMJZdcMqXYddddV9QvrhSuuOKKoj7pvsmrrz01yV95KQKKwFQEdAA4FRNNUQTGEoGXv/zl5qabbjLbbbddJfu/853vmM997nMTZTEAPOaYY1oZAIL3iSeeaDAInDat7MbOP/98c8opp5jjjjvOfOUrXzHf/va3zUknnTShDyKHH364edWrXmU++tGPdqXjZssttzRvf/vbzRFHHDElb1QTQgPAt7zlLUVbwJUCBoCoTw2KgCIwGgi8ZDTMUCsUAUWgLgJLLrmk2XDDDSuzefOb31y5bG7Bf//3fzcve9nLzPve976Jopdffrn58Ic/PDELeffdd5vvfve75qijjipoMLjFDNovf/nLKbOXxORTn/qU+dCHPmQ+//nPm1mzZlHy2F2XXXbZWm1h7ABTgxWBIURAZwCHsNJU5fFBILYsi6VkLJ1RoGW022+/3Xzwgx80yy23nFl++eXNgQceaF544QXzu9/9zrzrXe8yyyyzTLHMjBk0HkKyMLB605veZDBAXGuttczJJ5/Mi03E+RIwlg7XX3/9Ig+zbdCV9MVMHeIYkLkBM3rTp083mOELhUWLFpn//M//NLvsssvE7B9on332WfPSl750ohiWc5GG8Pzzz5u99trLfPaznzWvfvWrJ2jcyA477FAsA5977rluVtf9l770pcKGe+65pysdN4cddphZYoklzGOPPVbkYcCJpX0sSwNDLEFjlvWPf/zjlLJNJMDmgw46qKgzagMbbbSRufTSS7vYow7++te/mm984xsT9TNnzpyCxl0CxtI+ZlURqC5xRZvBH+IYXLsB6byNIl/anrCkf+aZZxZ2zJgxw/zDP/yD+cAHPmB+//vfu2L0XhFQBCogoAPACqBpEUWgLgJ/+9vfikEZBmb8Dy+9umHHHXc0b3zjG82CBQvMnnvuaU477TRzwAEHmPe+973FwANLte985zuLgcrFF18cFQe/uPe85z3FoPFb3/pWsaR60UUXma9//evRclg6JBrMwGGwh79P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SIAESKALAlwJpAtY/qStVsuJWrZsWefUCjN9/fLlywVxAwcO7MQ3dYc8vDVPHn08yKKPBfbIgzy8BLxHvD/6eKTBQn+39Xe8L/fm7NEA7KGuX3/9defq0aNH95ALLyUBEiABEiABEiiCAH7H11lnnSKKLrzMVYz1227GKlyU6gmwcuVKeeGFF2SttdaSVVZZxVEAXxUwCJcsWSJrr7129ZRKWWLy8AIljz4eZNHHAnvkQR5eAt4j3h99PNJgAdMHxt/IkSNlwIBmesOxBbDvnup6DzfNxhtvHHgdjD8agH1oyKOPBfbIo48HWfSx4L3hZUEe5NGfQF9Mr++Oprb8KcFmmr2qPbckQAIkQAIkQAIk0EACNAAbWOlUmQRIgARIgARIoNkEVj3ThGYjSF/7VVddVSZMmCCrrcYedtAlD+89Rh59PMiijwX2yIM8vAS8R7w/+niQRR+LpHscBJKUHK8jARIgARIgARIggYoSYBdwRSuOYpMACZAACZAACZBAUgI0AJOS43UkQAIkQAIkQAIkUFECNAArWnEUmwRIgARIgARIgASSEqABmJQcryMBEiABEiABEiCBihKgAZhyxd1yyy3yyU9+UlZffXVZf/315XOf+5ynhP/93/+V/fffX4YOHeqcP+GEE+Tdd9/1pKnbwTvvvCPbbLONs1rK448/7lHviSeekPHjxzu8Ro0aJWeffbbUbXGa5557TqZOnSqbbrqpo+dmm20mZ5xxRr96bwILd+XPnDnTYTJkyBDZdttt5be//a37dC33zzvvPNl+++2d1YM23HBDmTx5svzxj3/06Irn5fjjj3feD3hPTJo0SZ5//nlPmroegA9WVfr617/eUbFpPP7yl7/IEUccIcOHD5c11ljDeXc++uijHR54P2LyDqxggd8ZzDjx3//9353zddp577335Nvf/nbn3fmRj3zE+Y3AKlwamsRDdU5ta+AxpETg+uuvb6233nqt73//+y3zUm/9z//8T+tnP/tZJ3dzM7e22mqr1q677tp67LHHWnfccUfLPMSt4447rpOmjjvGyG195jOfwZKDrd///vcdFV977bXWRhtt1PrCF77QMsZP64YbbmiZZfVaF110USdNHXZ+9atftaZMmdK6/fbbW88880zr5ptvbpkf/9a0adM66jWFhSp83XXXtQYOHNi68sorW3/4wx9aJ554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tf5h3jcYv0F/7jpuAgu3vpdffrkz55kZ9NAyfk8tnQrFnaZu+0H3AOJwj2h46623nPcC3g94T+y3334t40aip2u/9RuATeMxf/58Z75Y9AxhaiTjLuKpc7R6mYnknZZApNlll10cQ9CTqCYHaCFGwwrmyjQ+oi3jUtUyfvaeXqMm8Ui7WldBhuYFxEACJEACJEACJEACJNAQAhwF3JCKppokQAIkQAIkQAIkoARoACoJbkmABEiABEiABEigIQRoADakoqkmCZAACZAACZAACSgBGoBKglsSIAESIAESIAESaAgBGoANqWiqSQIkQAIkQAIkQAJKgAagkuCWBEiABEiABEiABBpCgAZgQyqaapIACZAACZAACZCAEqABqCS4JQESaCwBrOd92WWXZaL/hAkTxKx0kknezJQESIAEkhKgAZiUHK8jARIohMCUKVNk8uTJicr+4Q9/KGb96X7XPvzww/Kv//qvnfhVVllFfv7zn3eOuUMCJEACdSOwWt0Uoj4kQAIk0C2BDTbYoNtLmJ4ESIAEKk2ALYCVrj4KTwIk4CZwySWXyNixY2Xo0KEyevRo+drXviZvvPGGk2ThwoVy1FFHyWuvvSZo4cOfWXvaOefuAsY+wmc/+1knjR4HtTyiaxddvBrefPNN+Zd/+RdZc8015UMf+pBcfPHFeqqzNWuGi1nMXkaNGuXI+clPflIgGwMJkAAJ5EmABmCetFkWCZBApgQGDBgg3/3ud+XJJ5+UH/3oR3L33Xc7xhYK3WmnnRw/v7XXXltefPFF5+/kk0/uJw+6gxHmzJnjpNHjfgkDIk455RRZsGCB3HTTTfLrX//aMeweffRRT0oYoffdd59cd911smjRIjnooINk7733lqefftqTjgckQAIkkCUBdgFnSZd5kwAJ5ErAPdhi0003lf/4j/+Qr371qzJz5kwZNGiQrLPOOk6r3ogRI0Ll0u5g+ApGpfNngJbG2bNny49//GPZY489nNMwQjfeeONO0meeeUauvfZaef7552XkyJFOPIzQ2267zTE4zz333E5a7pAACZBAlgRoAGZJl3mTAAnkSgCtbzCi/vCHP8iyZcvkvffek7ffflvQNYtu4SwDjDt07+64446dYoYNGyYf+9jHOsePPfaYtFot2WKLLTpx2HnnnXdk+PDhnjgekAAJkECWBGgAZkmXeZMACeRG4M9//rPss88+cswxxzgtfzC+7r33Xpk6daqsWLGiZznQvQzjzR3c+frPudPp/sqVK2XVVVcVdAtj6w7wG2QgARIggbwI0ADMizTLIQESyJTAI4884rT4YeAFjDWEn/70p54y0Q38/vvve+KCDgYOHNgvHbqG4VvoDo8//rggLcLmm2/u7D/44IPy4Q9/2Il79dVX5amnnpLx48c7x+PGjXPyfemll+TTn/60E8d/JEACJFAEAQ4CKYI6yyQBEuiJAEbywvhy/8FAQ5fvjBkz5Nlnn5W5c+fKrFmzPOVgRC989e666y55+eWXZfny5Z7zeoB0SLN06VKBEYew2267CYxM+PhhwMYZZ5zhMQjRgofWRgwEwbUwFjFyWI1R5IGu38MPP9wZKXzjjTfK4sWLBYNMLrjgArn11luRhIEESIAEciFAAzAXzCyEBEggTQKYNgWtae6/q666SjANDIyprbbaSq655ho577zzPMViJDC6iA855BCBwTh9+nTPeT1AK+Idd9zhTCWDMhD22msv+bd/+zdnVPH2228vr7/+umPI6TXYXnjhhbLLLrvIpEmTZPfdd5edd95Ztt12W3cSZ7AHpoqZNm2a4x+ItL/73e+csjwJeUACJEACGRJYxfiteJ1aMiyMWZMACZAACZAACZAACRRPgC2AxdcBJSABEiABEiABEiCBXAnQAMwVNwsjARIgARIgARIggeIJ0AAsvg4oAQmQAAmQAAmQAAnkSoAGYK64WRgJkAAJkAAJkAAJFE+ABmDxdUAJSIAESIAESIAESCBXAjQAc8XNwkiABEiABEiABEigeAI0AIuvA0pAAiRAAiRAAiRAArkSoAGYK24WRgIkQAIkQAIkQALFE6ABWHwdUAISIAESIAESIAESyJUADcBccbMwEiABEiABEiABEiieAA3A4uuAEpAACZAACZAACZBArgRoAOaKm4WRAAmQAAmQAAmQQPEE/j+uIoLshn+0QgAAAABJRU5ErkJggg==" width="640">


## Cloudiness (%) vs. Latitude


```python
plt.scatter(weather["latitude"], weather["cloudiness"], marker="o", color = 'blue')

# # Incorporate the other graph properties
plt.title("Cloudiness (%) vs. Latitude ")
plt.ylabel("Cloudiness (%)")
plt.xlabel("Latitude")
plt.grid()


plt.show()
```


    <IPython.core.display.Javascript object>



<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAoAAAAHgCAYAAAA10dzkAAAAAXNSR0IArs4c6QAAQABJREFUeAHsnQfcHUW5/58kBJIICRJ6EgFDkS4XkB6CFCGUKE1BEMSLePkrRBSQIkWRqhAQgQuIYKFcE4gIkWKhGZSmqKh0IZSQACGhRBLI/ue3xznvnHlnd2fr2XPObz6f8+7u7DNP+c7sOfPuzswOCFQSJhIgARIgARIgARIggZ4hMLBnImWgJEACJEACJEACJEACIQF2ANkQSIAESIAESIAESKDHCLAD2GMVznBJgARIgARIgARIgB1AtgESIAESIAESIAES6DEC7AD2WIUzXBIgARIgARIgARJgB5BtgARIgARIgARIgAR6jAA7gD1W4QyXBEiABEiABEiABNgBZBsgARIgARIgARIggR4jwA5gj1U4wyUBEiABEiABEiABdgDZBkiABEiABEiABEigxwiwA9hjFc5wSYAESIAESIAESIAdQLYBEiABEiABEiABEugxAuwA9liFM1wSIAESIAESIAESYAeQbYAESIAESIAESIAEeowAO4A9VuEMlwRIgARIgARIgATYAWQbIAESIAESIAESIIEeI8AOYI9VOMMlARIgARIgARIgAXYA2QZIgARIgARIgARIoMcIsAPYYxXOcEmABEiABEiABEiAHUC2ARIgARIgARIgARLoMQLsAPZYhTNcEiABEiABEiABEmAHkG2ABEiABEiABEiABHqMADuAPVbhDJcESIAESIAESIAE2AFkGyABEiABEiABEiCBHiPADmCPVTjDJQESIAESIAESIAF2ANkGSIAESIAESIAESKDHCLAD2GMVznBJgARIgARIgARIgB1AtgESIAESIAESIAES6DEC7AD2WIUzXBIgARIgARIgARJgB5BtgARIgARIgARIgAR6jAA7gD1W4QyXBEiABEiABEiABNgBZBsgARIgARIgARIggR4jwA5gj1U4wyUBEiABEiABEiABdgDZBkiABEiABEiABEigxwiwA9hjFc5wSYAESIAESIAESIAdQLYBEiABEiABEiABEugxAuwA9liFM1wSIAESIAESIAESYAeQbYAESiLwl7/8RT7/+c/LGmusIUOGDJGll15a/uu//kvOPfdcef3115tWx48fL/i0I5122mkyYMCAFtOrr766HHrooS15nXiw4447ype+9KWm63PnzpUDDjhAPvjBD8qHP/xhufzyy5vn9M4f//hHGTp0qPzjH//QWc3tN7/5zbD+Fi9e3Myr2w7qDe2siPTSSy8J2sef//znfupc7eaSSy6Rq6++up9sERntvEaK8J86SKCOBNgBrGOt0KeOJ3DFFVfIpptuKg8++KAce+yxctttt8lNN90k++23n1x22WXyhS98obYxwk90djo5/eIXv5Df//73LXF87Wtfkz/96U/y05/+VL7yla/I//zP/8i9997bDPO9996TL37xi3LcccfJuuuu28zXO1//+tfl2WeflWuuuUZndfUWHcDTTz/d2QH87//+b7n//vtb4i+zA9hiiAckQAKFEFiiEC1UQgIk0CSAH0Z0LnbeeWeZNm2aLLXUUs1zyENHBB3CuqZNNtmkrq55+3XmmWfKpz71KRk1alSzzK233iqTJ0+W3XffPfz86le/EuRtt912ocx3v/tdeffdd+XEE09sljF3RowYIQcddJCcffbZ4R1S+86pKdvt+6NHjxZ8mEiABDqXAO8Adm7d0fOaEkDnA50DPGI0O3/a3SWXXFL22msvfejc4hHxkUceGXZgII9HlieddFLYQdEF/vWvf4V2XI/dYB+P6cyEzs5HP/rR0Cc8lkaHx5XsR8B33XVXaOe6664LfVh11VVl+PDhstNOO8njjz/eT8Wvf/1rweNXyAwbNky22WYb+c1vftMiN2fOnPBu25gxY0J/VlhhhVAOZXXC3bo99thDVlxxxVAGdtF5e+GFF7SIc4tyDzzwgBx88MEt5//973/LBz7wgWYeHpUiD+mZZ56Rb3/72/K///u/zjrThaDziSeekN/97nc6y7n95Cc/Kauttpq4HhdvscUW4aNkXfDnP/+5IA8dTPBCXR922GH6dOHbp556KhyasNZaa4X20Enec8895a9//WvTFup88803D48xjAHtyWxT9iNgtJnHHntM7r777qYs8pDQPlEW7dVMul1hq1MQBOEQCbDDsAkMmUBH3ZXmz58vuCuLtoxrBHFMmjRJ3n77bZc480iABCwCvANoAeEhCeQh8P7778tvf/vb8PEvOjdZEjolO+ywgzz99NPhI7iNNtoofFR51llnhY/j0JFLm9ABmzhxomy11VZy/fXXC/zEWMRXXnnFWxXujKEzd+WVVwp+fI8//viw44DxcoMG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vfKdnZZ1p8pX+gj4nnvuaS4Dc9NNN4UdP6wHiCVgzjjjjE5lSL9JgARIgARIgARIoKMIVNoBxKPS5ZZbLgSEMYC44zdMvQ1+9913lyeffLKjwNFZEiABEiABEiABEuhUApV2AMeoEdT333+/vP322+EkkF122SXkNnfuXDUGaEinMqTfJEACJEACJEACJNBRBCodAzhJvYTxs5/9rCy99NLyoQ99SI0BGx/CwqPhDTfcsKPA0VkSIAESIAESIAES6FQClXYAjzzySPnYxz4WLgS98847h28BAbgPf/jDHAPYqS2IfpMACZAACZAACXQcgUo7gKCDmb8bbbSRPPvsszJ27FhZYoklwjGAHUeODpMACZAACZAACZBAhxKodAwg3gX8hS98IZz4sf7664eLQoPbUUcdJWeffXaHIqTbJEACJEACJEACJNBZBCrtAJ5wwgny6KOPyl133dUy6WOnnXaSG264obPI0VsSIAESIAESIAES6FAClT4CnjZtWtjR23LLLdU7P/teBrveeuvJ008/3aEI6TYJkAAJkAAJkAAJdBaBSu8AzpkzR1ZcccV+hLAsjNkh7CfADBIgARIgARIgARIggcIIVNoB3HzzzeXWW29tOq87fVdccYVstdVWzXzukAAJkAAJkAAJkAAJlEeg0kfAZ511luy6667y97//Xd577z258MIL5bHHHgsXh7777rvLi5KaSYAESIAESIAESIAEmgQqvQO49dZby+9//3vBbGAsAXPHHXfISiutFHYAN91006ZT3CEBEiABEiABEiABEiiPQKV3ABEG3vhxzTXXlBcRNZMACZAACZAACZAACcQSqLwDuHjxYnnqqadk9uzZgn0zjRs3zjzkPgmQAAmQAAmQAAmQQAkEKu0A/uEPf5ADDzxQnnvuOQmCoCUcTAh5//33W/J4QAIk0H4CuCzvvVfk5ZdFVllFZLvtRAYNivfLLqNGf8iMGa06tIb77hOZNStat63Lx77WXebW9EsvbqD+r3UyMmVNhmb+yisX7+3ChSKXXCJqmS1Rw25E1Ns4Zckl/eyYvpk+26V95exyaY+rspPWryLleyHGInlRV04CqiNWWdp4442D/fbbL1CTQIK5c+cGb7zxRsunMkcKMjRv3jz0YgNsdVq4cGGg1jsMsGUKQg7k0dcSOq19TJ0aBKNH47+1vg+OkR+VXGUG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BEiABEiABHJNgB3AXBcvjSMBEiABEiABEiCBSgLsAFYyoQ8JkAAJkAAJkAAJ5JoAO4C5Ll4aRwIkQAIkQAIkQAKVBNgBrGRCHxIgARIgARIgARLINQF2AHNdvDSOBEiABEiABEiABCoJsANYyYQ+JEACJEACJEACJJBrAuwA5rp4aRwJkAAJkAAJkAAJVBLoWelFn6QEzF6AflSsKK4OK5SvXbvWbGH0etttBac2ZHkkj3Ka5FHiQRYlFjgjD/IoJ1B+xfpR4pEFC31u63O8JL1zztgBrKGs38CGkcaNGDGiBilMSgIkQAIkQAIk0AwCeI7379+/GVk3PU/uBVxDEbxjdjZ/wewo/q53vUu6devmS8KvCnQIV5gd1vv161eD9HwkJY/yciSPEg+yKLHAGXmQRzmB8ivWjxKPLFjgzR86f0OHDpXu3TtzNhzfAJbqVOozVJrhw4c706Hzxw5gCQ15lFjgjDxKPMiixIJ1o5wFeZBHJYGST61tR6e++VOCndntVet5JAESIAESIAESIIEOJMAOYAcWOk0mARIgARIgARLobAI9zjeusxFkb32PHj1k3Lhx0rMnR9hBlzzK6xh5lHiQRYkFzsiDPMoJlF+xfpR4kEWJRbVn/AikWnJMRwIkQAIkQAIkQAJtSoBDwG1acFSbBEiABEiABEiABKolwA5gteSYjgRIgARIgARIgATalAA7gG1acFSbBEiABEiABEiABKolwA5gteSYjgRIgARIgARIgATalAA7gBkX3K233ip77bWX9O7dW7bccks58sgjy3L45z//KYcddphsscUWfvgXv/hF2bBhQ1mcvF2sX79ePvShD/m7pTz88MNl5v3tb3+T/fbbz+c1bNgw+eY3vyl525vx2WeflZNOOkm23357384ddthBzjvvvIpy7wQWduHPmjXLZ7L55pvL6NGj5Z577rGDc3l+0UUXyZ577unvHrTVVlvJEUccIU888USZrbhfzjzzTL99QDtx+OGHy8qVK8vi5PUCfLCr0tSpU4smdhqP559/Xj796U/LwIEDpU+fPn7buWzZsiIPtI9YvAM7WOA5gxUn/v73vxfD83SyadMm+drXvlZsO9/73vf6zwjswqWuk3iozZkdDTy6jAjccsst3nve8x7vyiuv9Eyj7j3++OPeL37xi6J0U5m9UaNGefvvv7/30EMPeYsW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BTDqENpLtJtoP/O6DIx5y+c/U3UZGNhsRhC8L3/5yx3Jo2h0RifsAGYEEmLwxg9ru2GNN6xnZ75o89f7srPAmzF82m4W8PTw68YMI5Ytg2LHzdO5qwMI+8wcKG+fffbx34Lil62ZV5e7t39Y9w2dX9efXcadwMK294orrvDXPDMfPXhm3pOnS6HYcfJ27qoD8EMdUffWW2/57QLaB7QTH//4xz0zjUSDc38MdgA7jcdvfvMbf71YjAxhaSQzXaSszPHWyywk778JRJx9993X7wiWRcrJBd4Q48UK1so0c0Q9M6XKM/Psy0aNOolH1sXaDQJNA0RHAiRAAiRAAiRAAiTQIQT4FXCHFDTNJAESIAESIAESIAElwA6gkuCRBEiABEiABEiABDqEADuAHVLQNJMESIAESIAESIAElAA7gEqCRxIgARIgARIgARLoEALsAHZIQdNMEiABEiABEiABElAC7AAqCR5JgARIgARIgARIoEMIsAPYIQVNM0mABEiABEiABEhACbADqCR4JAES6FgC2M975syZdbF/3LhxYnY6qYtsCiUBEiCBagmwA1gtOaYjARJoCoHJkyfLEUccUVXeP/nJT8TsP12R9s9//rN87nOfK/p369ZNFixYULzmCQmQAAnkjUDPvBlEe0iABEggLYFBgwalTcL4JEACJNDWBPgGsK2Lj8qTAAnYBC655BLZeeedZYsttpARI0bI6aefLm+++aYfZfHixXLCCSfImjVrBG/48Gf2nvbD7CFgnMN94hOf8OPotevNI4Z2McSr7j//+Y985jOfkb59+8qQIUPk+9//vgYVj2bPcDGb2cuwYcN8Pffaay+BbnQkQAIk0EgC7AA2kjbzIgESqCuB7t27y2WXXSaPPvqoXHfddXLXXXf5nS1kOmbMGH+eX79+/WTVqlX+35e+9KUKfTAcDDdnzhw/jl5XRHR4TJs2Tf7whz/IL3/5S/n973/vd+yWLVtWFhOd0Pvuu0/mzp0rjzzyiBxzzDFyyCGHyFNPPVUWjxckQAIkUE8CHAKuJ13KJgESaCgB+2OL7bffXr71rW/JaaedJrNmzZJevXpJ//79/bd6gwcPDtVLh4MxVzAqXlAA3jTOnj1brr/+ejnooIP8YHRChw8fXoz69NNPy8033ywrV66UoUOH+v7ohN52221+h3P69OnFuDwhARIggXoSYAewnnQpmwRIoKEE8PYNnajHHntMXn/9ddm0aZOsW7dOMDSLYeF6OnTuMLy79957F7MZMGCAvP/97y9eP/TQQ+J5nrzvfe8r+uFk/fr1MnDgwDI/XpAACZBAPQmwA1hPupRNAiTQMALPPfecTJw4UU499VT/zR86X/fee6+cdNJJsnHjxpr1wPAyOm+2s+UGw+x4ev7OO+9Ijx49BMPCONoO8wbpSIAESKBRBNgBbBRp5kMCJFBXAg8++KD/xg8fXqCzBvfzn/+8LE8MA7/99ttlfq6Lrq6uingYGsbcQts9/PDDgrhwI0eO9M+XLl0q22yzje/36quvypNPPin77beff73bbrv5clevXi377LOP78d/JEACJNAMAvwIpBnUmScJkEBNBPAlLzpf9h86aBjyvfzyy2X58uVyww03yFVXXVWWD77oxVy9O++8U15++WVZu3ZtWbheIB7ivPjii4JOHNz48eMFnUzM8cMHG+edd15ZhxBv8PC2ER+CIC06i/hyWDujkIGh3+OPP97/Unj+/PnyzDPPCD4yufjii2XhwoWIQkcCJEACDSHADmBDMDMTEiCBLAlg2RS8TbP/rr32WsEyMOhMjRo1Sm688Ua56KKLyrLFl8AYIj722GMFHcYZM2aUhesF3iIuWrTIX0oGecBNmDBBvv71r/tfFe+5557yxhtv+B05TYPjd7/7Xdl3333l8MMPlwMPPFDGjh0ro0ePtqP4H3tgqZizzz7bnx+IuA888ICfV1lEXpAACZBAHQl0M/NWyie11DEziiYBEiABEiABEiABEmg+Ab4BbH4ZUAMSIAESIAESIAESaCgBdgAbipuZkQAJkAAJkAAJkEDzCbAD2PwyoAYkQAIkQAIkQAIk0FAC7AA2FDczIwESIAESIAESIIHmE2AHsPllQA1IgARIgARIgARIoKEE2AFsKG5mRgIkQAIkQAIkQALNJ8AOYPPLgBqQAAmQAAmQAAmQQEMJsAPYUNzMjARIgARIgARIgASaT4AdwOaXATUgARIgARIgARIggYYSYAewobiZGQmQAAmQAAmQAAk0nwA7gM0vA2pAAiRAAiRAAiRAAg0lwA5gQ3EzMxIgARIgARIgARJoPoH/B1H9461GLuE+AAAAAElFTkSuQmCC" width="640">


## Wind Speed (mph) vs. Latitude


```python
plt.scatter(weather["latitude"], weather["windspeed"], marker="o", color = 'blue')

# # Incorporate the other graph properties
plt.title("Wind Speed (mph) vs. Latitude ")
plt.ylabel("Wind Speed (mph)")
plt.xlabel("Latitude")
plt.grid()


plt.show()
```


    <IPython.core.display.Javascript object>



<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAoAAAAHgCAYAAAA10dzkAAAAAXNSR0IArs4c6QAAQABJREFUeAHsnQfcFNW5/x9UQFBBEcX6D16JXgsYa2LHAsaCmmgsXFGM0ZjY27UrGo01lqsJ9i5WNGpUvGjAbq49thhrrGisKCKg7v/8Zn32PTvvlDO7M7uzM7/z+ezOzJlTv+fMnGee03pUjBEaEiABEiABEiABEiCB0hCYqzQ5ZUZJgARIgARIgARIgAQ8AhQAWRFIgARIgARIgARIoGQEKACWrMCZXRIgARIgARIgARKgAMg6QAIkQAIkQAIkQAIlI0ABsGQFzuySAAmQAAmQAAmQAAVA1gESIAESIAESIAESKBkBCoAlK3BmlwRIgARIgARIgAQoALIOkAAJkAAJkAAJkEDJCFAALFmBM7skQAIkQAIkQAIkQAGQdYAESIAESIAESIAESkaAAmDJCpzZJQESIAESIAESIAEKgKwDJEACJEACJEACJFAyAhQAS1bgzC4JkAAJkAAJkAAJUABkHSABEiABEiABEiCBkhGgAFiyAmd2SYAESIAESIAESIACIOsACZAACZAACZAACZSMAAXAkhU4s0sCJEACJEACJEACFABZB0iABEiABEiABEigZAQoAJaswJldEiABEiABEiABEqAAyDpAAiRAAiRAAiRAAiUjQAGwZAXO7JIACZAACZAACZAABUDWARIgARIgARIgARIoGQEKgCUrcGaXBEiABEiABEiABCgAsg6QAAmQAAmQAAmQQMkIUAAsWYEzuyRAAiRAAiRAAiRAAZB1gARIgARIgARIgARKRoACYMkKnNklARIgARIgARIgAQqArAMkQAIkQAIkQAIkUDICFABLVuDMLgmQAAmQAAmQAAlQAGQdIAESIAESIAESIIGSEaAAWLICZ3ZJgARIgARIgARIgAIg6wAJkAAJkAAJkAAJlIwABcCSFTizSwIkQAIkQAIkQAIUAFkHSIAESIAESIAESKBkBCgAlqzAmV0SIAESIAESIAESoADIOkACJEACJEACJEACJSNAAbBkBc7sRhO4+eabpUePHnLDDTd0c7jKKqt49+65555u95ZddllZbbXVavaDBw+WsWPH1q7TOBk+fLjgF2dmzJghp512miC9/fr1kwUWWECQvh122EHuv//+OO9tvT916lSPMY4u5sQTT5QVV1xRvvvuOxfnqbrRtKLOxJkxY8bItttuG+esrfdRt1ZeeeVU0vDiiy/KuHHj5M033+wWHp4LPB+2+f3vfy9//vOfbavUzrN4FlNLHAMigTYSoADYRviMOn8E0AhCAJwyZUpd4j755BN57rnnZL755ut275133pHXX39dNtpoo5qfW2+9VY499tjadatOvv32Wxk5cqScfPLJsv3228tNN90kEFAOOugg+fzzz+XBBx9sVVIyj+e9996T008/XSAEzjVXvl9lEIbuvPNO+etf/5o5lzxEAAHwhBNOCBQA8Vzg+bBNlgKgHQ/PSYAEugjM03XKMxIggYEDB3paEL8GCpqzeeaZR/bYY49uAqAKi7YAuOqqq7YF5gMPPCCPPPKIXHbZZbL77rvX0rDZZpvJvvvu2xZNWS0RKZ+ce+65suCCC8rPf/7zlENOPzhoYH/605/KqaeeKhtvvHH6EXRQiGBBQwIk0H4C+f5sbj8fpqCEBCDIvfzyy/L+++/Xcg+BcM0115QttthCnnzySfniiy/q7s0999yy/vrr1+z83U7aXXjdddfJ0UcfLUsssYTXPbvpppt6cdU8mpNKpeJptn7wgx/IvPPO63Ut33333baT0POPP/7Yu7f44osHurE1ZVdccYWn7Zw8ebInLA4YMMDTcI4aNcrTaPoDuPfee2WTTTbx0t23b19Zd9115b777vM7k1deeUVGjx4tiy66qPTu3VtWWGEF+eMf/9jN3T/+8Q9PKEJYELz33nvvOq7dPFgWs2fPlksvvdSLx84TuhyhwT3jjDO8bnCUQ58+fQSa3X/+858yZ84cOeKIIzz+/fv3l5/97Gfy4YcfWiGLwM9WW23laamGDRvmlcF//Md/yP/8z//UudMLhBlXpnCLbmAwfO2119Rr4BEfD3ZdUkfQ7i655JJ1Au/48eO9rv7555/f6+r/z//8TznqqKPUS+rHJ554QnbaaSePEbiC1c477yz/+te/anGhXv3iF7/wrvEsoTzwgz2MvwsY9zBs4corr6y5RXnBQHOK+36jddfuYkY5/Pd//7cstthigjq13nrryf/93//5vXrX06ZNk1//+tey1FJLSa9evWSZZZbxNJbffPNNoHtakkARCVAALGKpMk9NEVBNnq0FhJZvww039IQeNEh2VyruYfwfBIo4g8YZjeUll1wiF110kScs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F8KeLQTiGuacc/Ip/FVTx38SSIdAYQXADz74QMaMGSPQ7EHgw6LQEP4wzg/m7LPPlnnmmcfTAM6cOdPrEr7iiivMF18OP/nSKWuGQgIkQAIkYAhgiRcs9YLZwPb6/9D8QfjjEjCsJmUgUFgBEOv8RZl5551XzjvvPO8X5Y73SIAESIAEikcAQp7ZG0AefFCMokBk8cVF1l+fmr/ilTRzFEagsAJgWIZpTwIkQAIkQAIggA6f4cPJggTKSaAUC0GXs2iZaxIgARIgARIgARIIJkABMJgLbUmABEiABEiABEigsAQoABa2aJkxEiABEiABEiABEggmQAEwmAttSYAESIAESIAESKCwBCgAFrZomTESIAESIAESIAESCCZAATCYC21JgARIgARIgARIoLAEKAAWtmiZMRIgARIgARIgARIIJkABMJgLbUmABEiABEiABEigsAQoABa2aJkxEiABEiABEiABEggmQAEwmAttSYAESIAESIAESKCwBCgAFrZomTESIAESIAESIAESCCZAATCYC21JgARIgARIgARIoLAEKAAWtmiZMRIgARIgARIgARIIJkABMJgLbUmABEiABEiABEigsAQoABa2aJkxEiABEiABEiABEggmQAEwmAttSYAESIAESIAESKCwBCgAFrZomTESIAESIAESIAESCCZAATCYC21JgARIgARIgARIoLAEKAAWtmiZMRIgARIgARIgARIIJkABMJgLbUmABEiABEiABEigsAQoABa2aJkxEiABEiABEiABEggmQAEwmAttSYAESIAESIAESKCwBOYpbM6YMRIgARLoQALffivy8MMi778vsvjiIuuvLzL33B2YESa57QRQlx58kHWp7QWR0wRQAMxpwTBZJEAC5SQwdKjIq6925X2ppUTOPVfk5z/vsuMZCcQRuOUWkQMOEHnnnS6XrEtdLHgmwi5g1gISIAESyAGBO+6oJuLdd+sTg+vttxdBg05DAi4EUFdQZ2zhD/5Yl1zolccNBcDylDVzSgIkkFMC6Ko7/PDgxFUqVfsDDxSBOxoSiCKAOgLNn9Yb263asS7ZVMp7TgGwvGXPnJMACeSEAMZp+TV/dtLQcL/9dnU8l23PcxLwE0Bd8mv+bDesSzaNcp9TACx3+TP3JEACOSCACR8uxtWdS1h0U0wCrnXE1V0xKTFXIEABkPWABEiA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QBFQBBQBRWBIEdAh4CHNOBVbEVAEFAFFQBFQBBSBugioAlgXOQ2nCCgCioAioAgoAorAkCKgCuCQZpyKrQgoAoqAIqAIKAKKQF0EVAGsi5yGUwQUAUVAEVAEFAFFYEgRUAWw4Yz72te+Zrbcckuz8sorm9mzZ5t3vvOdHTH8+te/Nrvvvrt5yUte0nr/oQ99yCxbtqyDZtQeli5dajbffPPWaSl33XVXR/J+8pOfmDe/+c0tvObMmWM++clPmlE7nObBBx80Bx98sFl//fVb6dxwww3NggULJuX7OGBhZ/55553XwmSllVYyW2yxhfn3f/93+/VI3p966qnmDW94Q+v0oJe97GVmzz33NP/1X//VkVa+l7//+79v1Q/UE3vssYf5zW9+00Ezqg/gw6lKH/7whyeSOG54/Pd//7d5z3veY9ZYYw2zyiqrtOrOO+64YwIP6kc27+AEC9oZdpz42c9+NvF+lG5eeOEFc8IJJ0zUnRtssEGrjeAULnHjhIekubFrDp66hhD4l3/5l+ylL31pdv7552d5pZ7de++92TXXXDPBPS/M2aabbpptv/322Z133pndcMMNWf4RZ4cffvgEzSje5Epu9ta3vpUjB7Mf//jHE0l84oknsrXWWit717veleXKT3bttddm+bF62ac//ekJmlG4+frXv54deOCB2Te/+c3s/vvvz7785S9neeOfHXnkkRPJGxcsJMFXXnllNn369Ozzn/98ds8992T/8A//kOXKTvarX/1KSEbyussuu2QXX3xx9tOf/jTLO0PZ2972tuyVr3xl9vTTT0+kd/78+VneGWrVD9QT1Bevfe1rM+qPUXY/+MEPsrlz52Z/+Zd/2SoPktZxwuMPf/hDtt5667Xqi9tvvz174IEHsiVLlmS/+MUvBI7stNNOa9WT1JfUm/Pmzcte/vKXZ/nxaBM0o3Jz0kknZbkinH31q19tYUF7OnPmzOycc86ZSOI44TGR6IZusLaoawCB559/vlVp/9M//VOQ2/XXX5/lx8dleQ9vguaLX/xituKKK2YoAKPoSPOf//mfZ3kPdZICmFuAsvwonuy5556bSHpuAWgpxXkPb8JvFG/OOOOMLLcITiRt3LB44xvfmNGw245ycuyxx9peI3//6KOPtr6L7373u620/u///m9LMUZBFkd9Qb3xjW98Q7xG7pqfyZptvPHGLaU3HxGYUADHDY+PfvSj2TbbbBPMX+rFtddeu6UEChH1J/XoBRdcIF4jc6WDdNBBB3WkJx9Vy3ILactv3PDoAKKBBx0CbsiWmvfUDaZ7zgd+3eteZ/IemcmtXh2m+VtvvdXkFsCW6V6izS0ChiEO28Qv74b9+rvf/c4ceuih5rLLLmsNZbjpAQ+Gf3MFeOIVeDz88MOGYdNRdrnCb1ZfffWJJI4TFkx5oLzvvPPOE+nnhudbbrmlw2/UHygHOCkL4JJ3JjuwYaiPemOUsfm7v/s7kzf2Zscdd+zI8nHD49/+7d/M61//erPPPvsYpgjQluRW8glMcougeeSRRzrKB/Un9egolo9cGTY33nij+fnPf97C4D/+4z/MzTffbHbbbbfW87jhMVEQGrpRBbAhIH/5y1+2ODE3gzkLucna5MPBrQ8zN+u33vHh5kOeHTFCM2PGjNZH3fFiyB/yzonJhz1NbuVpVWi+5PjwEHx4N6ouHwY25557bgsbSeM4YfHYY4+ZP/3pT5O+BfJ+lPNd8lqufCNHHHGEoZFDwcORfuoD6gXbjTI2ubXT0IFm/p/rxg0P2pF8CpHJraEmnzLSqiOYJ37ppZe2oJHvQ+pJwWtUy0duETXvfve7TT46YPIpIy2FmPmh+OHGDQ/J76auqgCWIIlCx6Tk2O9HP/qRkUmpxx9/vNl7771bk9rzuT6tcPm8hYlY4OM6GgKfv0s3CM+peKDg5HNSzHHHHRcV2003WOBc/yiTKXqZioUtHtbNXXfdtdXDP+SQQ+xXk9I8TFh0JCTxwc3jYfoOEpMYJcvn/pq7777b5NNAonS8HFVsHnroIZPP/zSXX365YTFQqhtVPGhH/uqv/sqccsopLWXnAx/4QGsUBaXQduPy7Vx11VWtsrF48eJWJ+GSSy4x+Rxxw9V244KHneYm7ldogsko86CSzhcpRJOYT1w2+RyWFs0mm2wyQYtpnlVLrPzF5XM3TD6xd+I9N48//nhryMft0XUQDdBDKh755F1z2223dQzvkgyGN/bbb7/WBwwe0oOTJOZzolq3w4BHKhaSNpS/fEK/2WqrrcyFF14o3q3rsGPRkZiSB1bHc46nL++HId9Lkpf0mlW+DPd973vfM+uuu+5EGMoBQ+TUC7YVkO9i6623nqAblRuGeEkbq8DFYR0Gl89+9rMtK9g44cHUIbsNAZPXvOY1Jl/w0YKH8oHj24FWHBiO4rdz9NFHm3xe8EQbvNlmm5l8oVjLWnzAAQe02tRxwkPyu7Fr3pNS1wACLOJgMYe9CCSvuFqrPT/3uc+1YpBFILkiMBEjk70JN2qLQFjNyQo1+bECNi+0GSul815/K/0sfPizP/uzLJ8DOYEHK7pYGc3k3lFy+TYerUnurHj2reYcJyzIVxaBfPCDH+zI4ryhG/lFIJTrfL5bq4zn85o60s+DLHrILR8T76gvRnURCCtXpY6Qa95JbE3y53nc8MiHNictAsmHPLO809gqD5SfXAnMTj/99InyQf05qotA8rmxGXWj7XLraKsuxW/c8LBxaOJeVwE3geL/8WArC7ZvQNlhC5h877eWAsjSfpxsA/OWt7yltQ0My/vz3v/IbwND2h/ItzNAAbS3gaFyz3utGZUelf2XvvSlbNasWSO3DQyrODfaaKNshx12yFAEf/vb3078wAY3LlgUqc0y2Qbmoosuam0DQyPHNjD54h8hGckrSi+N9Xe+852JMkB5eOaZZybSy+po6gXqB7aBodyMwzYwAkC+oGFiFTB+44QHW+GssMIK2cknn5zdd9992RVXXJHlewFm+RC5wNNaAUwZor6k3qT+HNVtYHIrX6tNlW1gSHM+gpAdc8wxY4nHRKIbulEFsCEgYYPFj73d2OON/ezyFW2t/b7sKLCMsbQ938Azo3eTDyN2bINi047SvU8BJH35HKjsTW96U8sKSs82n1c3ctY/9n1D+fX97DweByzs9P7jP/5ja8+zfNFDls97ymQrFJtm1O59ZQA/yoi4Z599tlUvUD9QT7z97W/P8mkk8nrk/fpzDgAABoJJREFUr64COG54fOUrX2ntF8vIEFsj5dNFOvIcq1e+kXzLEgjNtttu21IEO4hG5AELMYYV9srM54hm+ZSqLJ9n3zFqNE54NJ2ty8Ewr4DUKQKKgCKgCCgCioAioAiMCQK6CnhMMlqTqQgoAoqAIqAIKAKKgCCgCqAgoVdFQBFQBBQBRUARUATGBAFVAMckozWZioAioAgoAoqAIqAICAKqAAoSelUEFAFFQBFQBBQBRWBMEFAFcEwyWpOpCCgCioAioAgoAoqAIKAKoCChV0VAEVAEFAFFQBFQBMYEAVUAxySjNZmKgCKgCCgCioAioAgIAqoAChJ6VQQUgbFFgPO8zznnnJ6kf7vttjP5SSc94a1MFQFFQBGoi4AqgHWR03CKgCIwJQgceOCBZs8996wV9z//8z+b/PzpSWF/+MMfmve///0T/sstt5y57rrrJp71RhFQBBSBUUNghVFLkKZHEVAEFIGqCKy55ppVgyi9IqAIKAJDjYBaAIc6+1R4RUARsBE466yzzGabbWZe8pKXmFe84hXmsMMOM08//XSL5Dvf+Y553/veZ5544gmDhY9ffvZ06509BMw9bq+99mrRyLPP8sjQLkO84v74xz+a9773vWbmzJnm5S9/uTnzzDPl1cQ1PzPc5IfZmzlz5rTk3HLLLQ2yqVMEFAFFoJ8IqALYT7Q1LkVAEegpAtOmTTOf+cxnzE9/+lNzySWXmJtuuqmlbBHp1ltv3ZrnN2vWLPPb3/629TvqqKMmycNwMO7iiy9u0cjzJEKPx9FHH22+/e1vm3/913813/rWt1qK3R133NFBiRL6/e9/31x55ZXm7rvvNvvss4/ZddddzX333ddBpw+KgCKgCPQSAR0C7iW6ylsRUAT6ioC92GL99dc3J554ovngBz9ozjvvPDNjxgyz2mqrtax6a6+9dlAuGQ5mrmCMzmWApfGiiy4yl156qdlpp51ar1FC11133QnS+++/33zxi180v/nNb8w666zT8kcJ/cY3vtFSOE855ZQJWr1RBBQBRaCXCKgC2Et0lbcioAj0FQGsbyhR99xzj3nyySfNCy+8YJ577jnD0CzDwr10KHcM72611VYT0ay++urm1a9+9cTznXfeabIsM6961asm/LhZunSpWWONNTr89EERUAQUgV4ioApgL9FV3oqAItA3BH71q1+Z3XbbzcyfP79l+UP5uvnmm83BBx9snn/++a7lYHgZ5c12Nl/3nU0n9y+++KJZfvnlDcPCXG3HvEF1ioAioAj0CwFVAPuFtMajCCgCPUXgRz/6Ucvix8ILlDXc1Vdf3REnw8B/+tOfOvx8D9OnT59Ex9Awcwttd9dddxlocRtttFHr/rbbbjOvfOUrW36PP/64+fnPf27e/OY3t55f97rXtfg++uij5k1velPLT/8UAUVAEZgKBHQRyFSgrnEqAopAVwiwkhfly/6hoDHke+6555pf/vKX5rLLLjMXXHBBRzys6GWu3o033mgee+wx88wzz3S8lwfooHnkkUcMShxuhx12MCiZzPFjwcaCBQs6FEIseFgbWQhCWJRFVg6LMgoPhn7322+/1krhL33pS+aBBx4wLDI5/fTTzfXXXw+JOkVAEVAE+oKAKoB9gVkjUQQUgSYRYNsUrGn27wtf+IJhGxiUqU033dRcccUV5tRTT+2IlpXADBHPmzfPoDCeccYZHe/lASviDTfc0NpKhjhwu+yyi/n4xz/eWlX8hje8wTz11FMtRU7CcP3Upz5ltt12W7PHHnuYHXfc0WyzzTZmiy22sElaiz3YKubII49szQ+E9vbbb2/F1UGoD4qAIqAI9BCB5fJ5K52TWnoYmbJWBBQBRUARUAQUAUVAEZh6BNQCOPV5oBIoAoqAIqAIKAKKgCLQVwRUAewr3BqZIqAIKAKKgCKgCCgCU4+AKoBTnwcqgSKgCCgCioAioAgoAn1FQBXAvsKtkSkCioAioAgoAoqAIjD1CKgCOPV5oBIoAoqAIqAIKAKKgCLQVwRUAewr3BqZIqAIKAKKgCKgCCgCU4+AKoBTnwcqgSKgCCgCioAioAgoAn1FQBXAvsKtkSkCioAioAgoAoqAIjD1CKgCOPV5oBIoAoqAIqAIKAKKgCLQVwRUAewr3BqZIqAIKAKKgCKgCCgCU4+AKoBTnwcqgSKgCCgCioAioAgoAn1FQBXAvsKtkSkCioAioAgoAoqAIjD1CPx//lHDqNqDe0MAAAAASUVORK5CYII=" width="640">



```python
Summary:

Mostly temperature went highest when the latitude reached between 20 to 40.
Average wind speed of 500 cities is 8 mph
Average humidity of 500 cities is 70%
```
