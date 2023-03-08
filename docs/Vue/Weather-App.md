# Vue 3 Practice - Weather App

Inspiration and boilerplate code for this project is from [AsmrProg - Weather App with Javascript](https://www.youtube.com/watch?v=iILFBGm_I9M&t=535s&ab_channel=AsmrProg) on Youtube. My goal for this exercise was to take this project and adapt it to a Vue 3 project using the Composition API for practice. 


## Demo
<video controls muted="true" autoplay>
	<source id="mp4" src="/assets/demo_weather_app_vue.mp4" type="video/mp4">
</video>

## Vue 3 Setup
For all my Vue 3 projects, Vue will be configured using Vite - [Vue 3 + Vite Setup](/vue/setup)

## API - WeatherAPI (Free)
1. Go to [WeatherAPI](https://www.weatherapi.com/) and create a free account
2. Once logged in, grab the API Key from your dashboard
	* should be a 31 character string similar to: `124f51569b574f5e939131853230703`
3. Head over to [/fields](https://www.weatherapi.com/my/fields.aspx) to edit the API fields received from the weather API response. Select the following fields in the section **Current Weather**:
	* `temp_c, text, code, wind_kph, humidity`

Your API json response should now look like
```json
{
    "location": {...},
    "current": {
        "temp_c": 14.0,
        "condition": {
            "text": "Light rain",
            "code": 1183
        },
        "wind_kph": 22.0,
        "humidity": 51,
    }
}
```

You can test your Weather API key for Paris by running the following command in your terminal
```
curl "http://api.weatherapi.com/v1/current.json?key={API_KEY}&q=Paris"
```

## Migrating from JS to Vue 3

### 1. Images
Images from `/images`  have been moved to the `public/` folder in the Vue 3 project

### 2. HTML
`index.html` has been refactored with the Composition API into a Vue component called `WeatherCard.Vue` at path `src/components/WeatherCard.Vue`. 

* Kept the Font Awesome JS script 
```html hl_lines="12"
<!-- index.html  -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Weather App in Vue 3</title>
  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="/src/main.js"></script>
    <script type="application/javascript" src="https://kit.fontawesome.com/7c8801c017.js" crossorigin="anonymous"></script>
  </body>
</html>
```

### 3. CSS
Refactored the `style.css` into `WeatherCard.Vue` component within the `<script scoped>` section as it increases reusability for the future. 

### 4. Javascript
`index.js` javascript code has been moved into `WeatherCard.Vue`'s `<script setup>` section to encapsulate the component's code logic into one place.

* Weather API used in the youtube video was no longer available, thus it was changed to the free [Weather API](https://www.weatherapi.com/) and extracted similar data as the tutorial.
```js
<script setup>
...
function get_weather() {
    const container = document.querySelector('.container');
    const weatherBox = document.querySelector('.weather-box');
    const weatherDetails = document.querySelector('.weather-details');
    const city = document.querySelector('.search-box input').value;

    const APIKey = '<YOUR-WEATHER-API-KEY>';

    if (city === '') {
        return;
    }
    
    fetch(`http://api.weatherapi.com/v1/current.json?key=${APIKey}&q=${city}`)
        .then(response => response.json())
        .then(weather => {
            console.log(weather);       // debug API values

            const image = document.querySelector('.weather-box img');
            const temperature = document.querySelector('.weather-box .temperature');
            const description = document.querySelector('.weather-box .description');
            const humidity = document.querySelector('.weather-details .humidity span');
            const wind = document.querySelector('.weather-details .wind span');

            if (weather.hasOwnProperty('error')) {
                temperature.style.display = 'none'
                weatherDetails.style.display = 'none';
                
                image.src = '/404.png';
                description.innerHTML = `${weather.error.message}`;
                container.style.height = '350px';

            } else {
                // weather card API values
                image.src = `/${weather_img(weather.current.condition.code)}.png`;
                
                temperature.innerHTML = `${parseInt(weather.current.temp_c)}<span>°C</span>`;
                temperature.style.display = ''
                description.innerHTML = `${weather.current.condition.text}`;
                
                humidity.innerHTML = `${weather.current.humidity}%`;
                wind.innerHTML = `${parseInt(weather.current.wind_kph)}Km/h`;

                weatherDetails.style.display = '';
                weatherDetails.classList.add('fadeIn');
                
                container.style.height = '600px'; 
            }

            weatherBox.style.display = '';
            weatherBox.classList.add('fadeIn');
        })
}
<script/>
```
* Using the API's [response codes](https://www.weatherapi.com/docs/weather_conditions.json), the logic determines which weather icon is displayed
```js
function weather_img(code) {
    // API codes - https://www.weatherapi.com/docs/weather_conditions.json
    if (code == 1000) {
        return 'clear'
    }
    else if (code === 1030) {
        return 'mist'
    }
    else if ([1003, 1006, 1009].includes(code)) {
        return 'cloud'
    }
    else if ([1063, 1180, 1183, 1186, 1189, 1192, 1995, 1198, 1201, 1240, 1243, 1246, 1273, 1276].includes(code)) {
        return 'rain'
    }
    else if ([1210, 1213, 1216, 1219, 1222, 1225, 1255, 1258].includes(code)) {
        return 'snow'
    }
    else {
        return ''
    }
}
```



### 5. Vue Components
**App.Vue**

```typescript hl_lines="2 6"
<script setup>
  import WeatherCard from './components/WeatherCard.vue';
</script>

<template>
  <WeatherCard/>
</template>
```
<br>

**WeatherCard.Vue**

* Replaced the `EventListener` to Vue's `@click` and bind the custom javascript function `get_weather` when users click on the search icon
```html hl_lines="6"
<template>
    <div class="container">
        <div class="search-box">
            <i class="fa-solid fa-location-dot"></i>
            <input type="text" placeholder="Enter your location">
            <button @click="get_weather" class="fa-solid fa-magnifying-glass"></button>
        </div>

        <!-- removed 'not-found' section -->

        <div class="weather-box">
            <img src="">
            <p class="temperature"></p>
            <p class="description"></p>
        </div>

        <div class="weather-details">
            <div class="humidity">
                <i class="fa-solid fa-water"></i>
                <div class="text">
                    <span></span>
                    <p>Humidity</p>
                </div>
            </div>
            <div class="wind">
                <i class="fa-solid fa-wind"></i>
                <div class="text">
                    <span></span>
                    <p>Wind Speed</p>
                </div>
            </div>
        </div>
    </div>
</template>
```

## Notes
The Weather API contains a lot more fields that can be consummed for this type of application. This project could be expanded by retrieving/displaying more weather information or saving multiple weather cards for locations and keeping information in real-time. 

[Read More - Weather API](https://www.weatherapi.com/api.aspx){ .md-button}


## Code

All the code used for this project is available in the github repo, open an issue if you have any questions or problems.

[Github Repo :simple-github:](#){ .md-button }

