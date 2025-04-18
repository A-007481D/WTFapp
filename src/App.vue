<template>
  <div class="container">
    <h1>What the Forecast?!</h1>

    <div class="search-box">
      <input
          v-model="city"
          @keyup.enter="getWeather"
          type="text"
          placeholder="Enter city name"
      />
      <button @click="getWeather">Search</button>
    </div>

    <button @click="getLocationWeather" class="location-btn">
      Use My Location
    </button>

    <div v-if="loading" class="message">
      Loading weather data...
    </div>

    <div v-else-if="error" class="error">
      {{ error }}
    </div>

    <div v-else-if="weather" class="weather-info">
      <h2>{{ weather.name }}, {{ weather.sys.country }}</h2>

      <div class="main-weather">
        <img
            :src="`https://openweathermap.org/img/wn/${weather.weather[0].icon}@2x.png`"
            :alt="weather.weather[0].description"
        />
        <div class="temp">{{ Math.round(weather.main.temp) }}°C</div>
      </div>

      <p class="description">{{ weather.weather[0].description }}</p>

      <div class="details">
        <div class="detail-item">
          <span>Feels like</span>
          <span>{{ Math.round(weather.main.feels_like) }}°C</span>
        </div>
        <div class="detail-item">
          <span>Humidity</span>
          <span>{{ weather.main.humidity }}%</span>
        </div>
        <div class="detail-item">
          <span>Wind</span>
          <span>{{ Math.round(weather.wind.speed * 3.6) }} km/h</span>
        </div>
        <div class="detail-item">
          <span>Pressure</span>
          <span>{{ weather.main.pressure }} hPa</span>
        </div>
      </div>

      <div v-if="forecast && forecast.list" class="forecast">
        <h3>Forecast</h3>
        <div class="forecast-items">
          <div
              v-for="(item, index) in forecast.list.slice(0, 5)"
              :key="index"
              class="forecast-item"
          >
            <div>{{ formatTime(item.dt) }}</div>
            <img
                :src="`https://openweathermap.org/img/wn/${item.weather[0].icon}.png`"
                :alt="item.weather[0].description"
            />
            <div>{{ Math.round(item.main.temp) }}°C</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const API_KEY = '4d3baec517736fbe8eb5f89f8e1c5778\n';
const BASE_URL = 'https://api.openweathermap.org/data/2.5';

export default {
  name: 'App',
  data() {
    return {
      city: '',
      weather: null,
      forecast: null,
      loading: false,
      error: ''
    }
  },
  methods: {
    async getWeather() {
      if (!this.city.trim()) return;

      this.loading = true;
      this.error = '';

      try {
        //current weather
        const weatherResponse = await fetch(
            `${BASE_URL}/weather?q=${this.city}&units=metric&appid=${API_KEY}`
        );

        if (!weatherResponse.ok) {
          if (weatherResponse.status === 404) {
            throw new Error('City not found. Please check the spelling and try again.');
          }
          throw new Error('Failed to fetch weather data');
        }

        this.weather = await weatherResponse.json();

        // forecast
        const forecastResponse = await fetch(
            `${BASE_URL}/forecast?q=${this.city}&units=metric&appid=${API_KEY}`
        );

        if (forecastResponse.ok) {
          this.forecast = await forecastResponse.json();
        }
      } catch (err) {
        this.error = err.message || 'Failed to fetch weather data';
        this.weather = null;
        this.forecast = null;
      } finally {
        this.loading = false;
      }
    },

    getLocationWeather() {
      if (!navigator.geolocation) {
        this.error = 'Geolocation is not supported by your browser';
        return;
      }

      this.loading = true;
      this.error = '';

      navigator.geolocation.getCurrentPosition(
          async (position) => {
            try {
              const { latitude, longitude } = position.coords;

              // current weather
              const weatherResponse = await fetch(
                  `${BASE_URL}/weather?lat=${latitude}&lon=${longitude}&units=metric&appid=${API_KEY}`
              );

              if (!weatherResponse.ok) {
                throw new Error('Failed to fetch weather data');
              }

              this.weather = await weatherResponse.json();

              const forecastResponse = await fetch(
                  `${BASE_URL}/forecast?lat=${latitude}&lon=${longitude}&units=metric&appid=${API_KEY}`
              );

              if (forecastResponse.ok) {
                this.forecast = await forecastResponse.json();
              }
            } catch (err) {
              this.error = err.message || 'Failed to fetch weather data';
              this.weather = null;
              this.forecast = null;
            } finally {
              this.loading = false;
            }
          },
          (err) => {
            this.loading = false;
            this.error = 'Unable to retrieve your location';
          }
      );
    },

    formatTime(timestamp) {
      const date = new Date(timestamp * 1000);
      return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
    }
  },
  mounted() {
    // default to agadir upon page load
    this.city = 'Agadir';
    this.getWeather();
  }
}
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: Arial, sans-serif;
  line-height: 1.6;
  background-color: #f5f5f5;
  color: #333;
}

.container {
  max-width: 500px;
  margin: 0 auto;
  padding: 20px;
  background-color: white;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  margin-top: 30px;
}

h1 {
  text-align: center;
  margin-bottom: 20px;
  color: #333;
}

.search-box {
  display: flex;
  margin-bottom: 15px;
}

input {
  flex: 1;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px 0 0 4px;
  font-size: 16px;
}

button {
  padding: 10px 15px;
  background-color: #4a90e2;
  color: white;
  border: none;
  border-radius: 0 4px 4px 0;
  cursor: pointer;
  font-size: 16px;
}

.location-btn {
  width: 100%;
  border-radius: 4px;
  background-color: #f0f0f0;
  color: #333;
  margin-bottom: 20px;
}

.message, .error {
  text-align: center;
  padding: 20px;
}

.error {
  background-color: #ffebee;
  color: #c62828;
  border-radius: 4px;
}

.weather-info {
  text-align: center;
}

.main-weather {
  display: flex;
  justify-content: center;
  align-items: center;
}

.temp {
  font-size: 48px;
  font-weight: bold;
}

.description {
  text-transform: capitalize;
  margin-bottom: 20px;
}

.details {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
  margin-bottom: 20px;
}

.detail-item {
  background-color: #f5f5f5;
  padding: 10px;
  border-radius: 4px;
  display: flex;
  flex-direction: column;
}

.detail-item span:first-child {
  font-size: 14px;
  color: #666;
}

.detail-item span:last-child {
  font-weight: bold;
}

.forecast h3 {
  margin-bottom: 10px;
}

.forecast-items {
  display: flex;
  overflow-x: auto;
  gap: 10px;
  padding-bottom: 10px;
}

.forecast-item {
  min-width: 80px;
  text-align: center;
}

.forecast-item img {
  width: 50px;
  height: 50px;
}
</style>