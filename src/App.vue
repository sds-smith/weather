<template>
  <div id="app" :class="weather.main !== undefined && weather.main.temp > 50 ? 'warm' : ''">
    <main>
      <h1>Weather App</h1>
      <div class="search-box">
        <input 
          type="text" 
          class="search-bar" 
          :style="{display : city_display}"
          :placeholder="city_placeholder"
          v-model="cityQuery"
          @focus="citySearch"
          @keyup.enter="fetchCityWeather"
        />
        <div class="city_details" :style="{display : city_details_display}">
          <select 
              name="country" 
              id="country" 
              class="search-bar" 
              v-model="country"
              @keyup.enter="fetchCityWeather"  
          >
            <option value="">Select Country</option>
          </select>
          <select 
              name="state" 
              id="state" 
              class="search-bar" 
              v-model="state" 
              :disabled="country === 'USA' ? false : true"
              @change="fetchCityWeather"    
          >
            <option value="">Select State</option>
          </select>
        </div>

        <input 
          type="text" 
          class="search-bar" 
          :style="{display : coordinates_display}"
          :placeholder="coordinates_placeholder"
          v-model="latitude"
          @focus="coordinatesSearch"
          @keyup.enter="focusInput"
        />
        <input 
          type="text" 
          class="search-bar" 
          ref="longitude"
          :style="{display : coordinates_details_display}"
          placeholder="Enter Longitude"
          v-model="longitude"
          @focus="coordinatesSearch"
          @keyup.enter="fetchCoordinatesWeather"
        />
      </div>

      <div class="weather-wrap" v-if="weather.main !== undefined">
        <div class="location-box">
          <div class="location">{{ weather.name }} {{geoLocation.state}}, {{ weather.sys.country }}</div>
          <div class="date">{{ dateBuilder() }}</div>
        </div>

        <div class="weather-box">
          <div class="temp">{{ Math.round(weather.main.temp) }}°f</div>
          <div class="weather">{{ weather.weather[0].description }}</div>
          <p >{{ weather.clouds.all }}% cloudy</p>
          <img :src="`https://openweathermap.org/img/wn/${weather.weather[0].icon}@2x.png`"/>
          <div class="more-info">
            <p >Humidity: {{ weather.main.humidity }}%</p>
            <p >Visibility: {{ weather.visibility }}m</p>
            <p >Wind: {{ weather.wind.speed }}mph {{ degToCompass(weather.wind.deg) }}</p>
            <p >Sunrise: {{ new Date(weather.sys.sunrise * 1000).toLocaleTimeString() }}</p>
            <p >Sunset: {{ new Date(weather.sys.sunset * 1000).toLocaleTimeString() }}</p>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import {countryListAlpha3, states} from './assets/geoCodes'

export default {

  mounted() {
    for (let country in countryListAlpha3) {
      const option = document.createElement('option')
      option.innerHTML = countryListAlpha3[country]
      option.value = country
      document.getElementById('country').appendChild(option)
    }

    states.forEach(state => {
      const option = document.createElement('option')
      option.innerHTML = state
      option.value = state
      document.getElementById('state').appendChild(option)
    })
  },

  name: 'app',

  data () {
    return {
      api_key: '20ffadd488fb7af5565937af674f68ae',
      url_base: 'https://api.openweathermap.org/data/2.5/', 
      city_placeholder: "Search by City...",
      city_display: 'block',
      city_details_display: 'none',
      coordinates_placeholder: "Search by Coordinates...",
      coordinates_display: 'block',   
      coordinates_details_display: 'none',   
      cityQuery: '',
      country: '',
      state: '',
      latitude: '',
      longitude: '',
      geoLocation: {},
      weather: {}
    }
  },

  methods: {
    citySearch() {
      this.coordinates_display = 'none'
      this.city_details_display = 'block'
      this.city_placeholder = "Enter City Name"
    },

    focusInput() {
      this.$refs.longitude.focus();
    },

    resetInputs() {
      this.city_placeholder = "Search by City...",
      this.city_display = 'block',
      this.city_details_display = 'none',
      this.coordinates_placeholder = "Search by Coordinates...",
      this.coordinates_display = 'block',   
      this.coordinates_details_display = 'none',
      this.cityQuery = '',
      this.country = '',
      this.state = '',
      this.latitude = '',
      this.longitude = '',
      document.activeElement.blur()
    },

    coordinatesSearch() {
      this.city_display = 'none'
      this.coordinates_details_display = 'block'
      this.coordinates_placeholder = "Enter Latitude"
    },

    async fetchCityWeather() {
        let endpoint
        if (this.country === 'USA') {
          endpoint = `https://api.openweathermap.org/geo/1.0/direct?q=${this.cityQuery},${this.state},${this.country}&limit=1&appid=${this.api_key}`
        } else {
          endpoint = `https://api.openweathermap.org/geo/1.0/direct?q=${this.cityQuery},${this.country}&limit=1&appid=${this.api_key}`
        }    
        const response = await fetch(endpoint)
        const jsonResponse = await response.json()
        console.log(jsonResponse)
        this.geoLocation = jsonResponse[0]
        this.latitude = jsonResponse[0].lat
        this.longitude = jsonResponse[0].lon
        this.fetchCoordinatesWeather()
    },

    async fetchCoordinatesWeather() {
      const response = await fetch(`${this.url_base}weather?lat=${this.latitude}&lon=${this.longitude}&units=imperial&appid=${this.api_key}`)
      this.weather = await response.json()
      console.log(this.weather)
      this.resetInputs()
    },

    dateBuilder() {
      let d = new Date();
      let months = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];
      let days = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
      let day = days[d.getDay()];
      let date = d.getDate();
      let month = months[d.getMonth()];
      let year = d.getFullYear();
      let time = d.toLocaleTimeString();
      return `${day} ${month} ${date} ${year}  ${time}`;
    },

    degToCompass(num) {
        const val = Math.floor((num / 22.5) + 0.5);
        const compassArr = ["N", "NNE", "NE", "ENE", "E", "ESE", "SE", "SSE", "S", "SSW", "SW", "WSW", "W", "WNW", "NW", "NNW"];
        return compassArr[(val % 16)];
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
body {
  font-family: 'trebuchet', sans-serif;
}

#app {

  background-color: aquamarine;
  background-image: linear-gradient(to bottom, rgba(0, 0, 0, 0.15), rgba(0, 0, 0, 0.65));
  transition: 0.4s;
}
#app.warm {
  background-color: aqua;
}
main {
  min-height: 100vh;
  width: 50%;
  margin: 0 auto;
  padding: 25px;
  background-image: linear-gradient(to bottom, rgba(0, 0, 0, 0.05), rgba(0, 0, 0, 0.45));
}
h1 {
  color: white;
  text-align: center;
  margin: 20px 0px;
}
.search-box {
  width: 100%;
  margin-bottom: 30px;
}
.search-box .search-bar {
  display: block;
  width: 100%;
  padding: 15px;
  margin: 10px 0px;
  color: #313131;
  font-size: 20px;
  appearance: none;
  border:none;
  outline: none;
  background: none;
  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.5);
  border-radius: 0px 16px 0px 16px;
  transition: 0.4s;
}
.search-box .search-bar:focus {
  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.75);
  border-radius: 16px 16px 16px 16px;
}

.city_details {
  width: 100%;
}

#country,
#state {
  width: 40%;
  display: inline-block;
  margin: 0px 5%;
}
.location-box .location {
  color: #FFF;
  font-size: 32px;
  font-weight: 500;
  text-align: center;
  text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
}
.location-box .date {
  color: #FFF;
  font-size: 20px;
  font-weight: 300;
  font-style: italic;
  text-align: center;
}
.weather-box {
  text-align: center;
}
.weather-box .temp {
  display: inline-block;
  padding: 10px 25px;
  color: #FFF;
  font-size: 102px;
  font-weight: 900;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
  background-color:rgba(255, 255, 255, 0.25);
  border-radius: 16px;
  margin: 30px 0px;
  box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}
.weather-box .weather {
  color: #FFF;
  font-size: 48px;
  font-weight: 700;
  font-style: italic;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

p {
  color: white;
  font-size: 1rem;
  font-weight: bold;
  margin: 5px 0px
}
@media screen and (max-width : 1020px) {
  main {
    width: 100%;
  }

  .location-box .location {
    font-size: 24px;
  }

  .weather-box .temp {
    font-size: 60px;
  }

  .weather-box .weather {
    font-size: 24px;
  }
}
</style>