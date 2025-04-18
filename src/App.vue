<template>
  <div class="container" :class="{ dark: isDarkMode }">
    <div class="theme-toggle">
      <label>
        <input id="mode-toggler" type="checkbox" v-model="isDarkMode" @change="toggleTheme" />
        {{ isDarkMode ? '🌙 Dark Mode' : '☀️ Light Mode' }}
      </label>
    </div>

    <h1>What the Forecast?!</h1>

    <div class="search-box">
      <input v-model="city" @keyup.enter="getWeather" type="text" placeholder="Enter city name" />
      <button @click="getWeather">Search</button>
    </div>

    <button @click="getLocationWeather" class="location-btn">
      Use My Location
    </button>

    <div v-if="loading" class="message">Loading weather data...</div>

    <div v-else-if="error" class="error">{{ error }}</div>

    <div v-else-if="weather" class="weather-info">
      <h2>{{ weather.name }}, {{ weather.sys.country }}</h2>

      <div class="main-weather">
        <img :src="`https://openweathermap.org/img/wn/${weather.weather[0].icon}@2x.png`"
          :alt="weather.weather[0].description" />
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
        <h3>Next Hours</h3>
        <div class="forecast-items">
          <div v-for="(item, index) in forecast.list.slice(0, 5)" :key="index" class="forecast-item">
            <div>{{ formatTime(item.dt) }}</div>
            <img :src="`https://openweathermap.org/img/wn/${item.weather[0].icon}.png`"
              :alt="item.weather[0].description" />
            <div>{{ Math.round(item.main.temp) }}°C</div>
          </div>
        </div>
      </div>

      <div v-if="dailyForecast.length > 0" class="daily-forecast">
        <h3>5-Day Forecast</h3>
        <div class="daily-forecast-items">
          <div v-for="(day, index) in dailyForecast" :key="index" class="daily-forecast-item">
            <div class="day">{{ formatDay(day.dt) }}</div>
            <img :src="`https://openweathermap.org/img/wn/${day.icon}.png`" :alt="day.description" />
            <div class="temp-range">
              <span class="max-temp">{{ Math.round(day.maxTemp) }}°</span>
              <span class="min-temp">{{ Math.round(day.minTemp) }}°</span>
            </div>
            <div class="day-description">{{ day.description }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const API_KEY = '4d3baec517736fbe8eb5f89f8e1c5778';
const BASE_URL = 'https://api.openweathermap.org/data/2.5';

export default {
  name: 'App',
  data() {
    return {
      city: '',
      weather: null,
      forecast: null,
      dailyForecast: [],
      loading: false,
      error: '',
      isDarkMode: false,
    };
  },
  methods: {
    async getWeather() {
      if (!this.city.trim()) return;
      this.loading = true;
      this.error = '';
      this.dailyForecast = [];

      try {
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

        const forecastResponse = await fetch(
          `${BASE_URL}/forecast?q=${this.city}&units=metric&appid=${API_KEY}`
        );

        if (forecastResponse.ok) {
          this.forecast = await forecastResponse.json();
          this.processDailyForecast(this.forecast.list);
        }
      } catch (err) {
        this.error = err.message || 'Failed to fetch weather data';
        this.weather = null;
        this.forecast = null;
      } finally {
        this.loading = false;
      }
    },

    processDailyForecast(forecastList) {
      const dailyData = {};
      forecastList.forEach(item => {
        const date = new Date(item.dt * 1000);
        const day = date.toISOString().split('T')[0];
        if (!dailyData[day]) {
          dailyData[day] = {
            temps: [],
            icons: {},
            descriptions: {},
            dt: item.dt,
          };
        }
        dailyData[day].temps.push(item.main.temp);
        const icon = item.weather[0].icon.replace('n', 'd');
        const description = item.weather[0].description;
        dailyData[day].icons[icon] = (dailyData[day].icons[icon] || 0) + 1;
        dailyData[day].descriptions[description] = (dailyData[day].descriptions[description] || 0) + 1;
      });

      this.dailyForecast = Object.values(dailyData)
        .map(day => {
          const mostCommonIcon = Object.entries(day.icons).sort((a, b) => b[1] - a[1])[0][0];
          const mostCommonDescription = Object.entries(day.descriptions).sort((a, b) => b[1] - a[1])[0][0];
          return {
            dt: day.dt,
            minTemp: Math.min(...day.temps),
            maxTemp: Math.max(...day.temps),
            icon: mostCommonIcon,
            description: mostCommonDescription,
          };
        })
        .sort((a, b) => a.dt - b.dt)
        .slice(0, 5);
    },

    getLocationWeather() {
      if (!navigator.geolocation) {
        this.error = 'Geolocation is not supported by your browser';
        return;
      }

      this.loading = true;
      this.error = '';
      this.dailyForecast = [];

      navigator.geolocation.getCurrentPosition(
        async position => {
          try {
            const { latitude, longitude } = position.coords;
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
              this.processDailyForecast(this.forecast.list);
            }
          } catch (err) {
            this.error = err.message || 'Failed to fetch weather data';
            this.weather = null;
            this.forecast = null;
          } finally {
            this.loading = false;
          }
        },
        err => {
          this.loading = false;
          this.error = 'Unable to retrieve your location';
        }
      );
    },

    formatTime(timestamp) {
      const date = new Date(timestamp * 1000);
      return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
    },

    formatDay(timestamp) {
      const date = new Date(timestamp * 1000);
      return date.toLocaleDateString([], { weekday: 'long' });
    },

    toggleTheme() {
      localStorage.setItem('darkMode', this.isDarkMode);
      document.body.classList.toggle('dark', this.isDarkMode);
    }
  },

  mounted() {
    this.city = 'Agadir';
    this.getWeather();

    const savedTheme = localStorage.getItem('darkMode');
    this.isDarkMode = savedTheme === 'true';
    this.toggleTheme();
  }
};
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
  transition: background-color 0.3s, color 0.3s;
}

body.dark {
  background-color: #121212;
  color: #eee;
}

.container {
  max-width: 500px;
  margin: 30px auto;
  padding: 20px;
  background-color: white;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  transition: background-color 0.3s, color 0.3s;
}

.dark .container {
  background-color: #1e1e1e;
  color: #eee;
}

h1 {
  text-align: center;
  margin-bottom: 20px;
  color: #333;
}

.dark h1 {
  color: #fff;
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
  background-color: white;
  color: #333;
  transition: background-color 0.3s, color 0.3s;
}

.dark input {
  background-color: #333;
  color: #eee;
  border-color: #555;
}

button {
  padding: 10px 15px;
  background-color: #4a90e2;
  color: white;
  border: none;
  border-radius: 0 4px 4px 0;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #357abd;
}

.location-btn {
  width: 100%;
  border-radius: 4px;
  background-color: #f0f0f0;
  color: #333;
  margin-bottom: 20px;
}

.dark .location-btn {
  background-color: #333;
  color: #eee;
}

.message,
.error {
  text-align: center;
  padding: 20px;
}

.error {
  background-color: #ffebee;
  color: #c62828;
  border-radius: 4px;
}

.dark .error {
  background-color: #4e0000;
  color: #ff8a80;
}

/* Weather Info */
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
  background-color: #f0f0f0;
  padding: 10px;
  border-radius: 6px;
}

.dark .detail-item {
  background-color: #2a2a2a;
}

.forecast,
.daily-forecast {
  margin-top: 20px;
}

.forecast-items,
.daily-forecast-items {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 10px;
}

.forecast-item,
.daily-forecast-item {
  flex: 1 1 90px;
  background-color: #f0f0f0;
  padding: 10px;
  border-radius: 6px;
  text-align: center;
}

.dark .forecast-item,
.dark .daily-forecast-item {
  background-color: #2a2a2a;
}

.day-description {
  font-size: 12px;
  text-transform: capitalize;
}

.temp-range {
  display: flex;
  justify-content: center;
  gap: 6px;
}

.max-temp {
  font-weight: bold;
}

.theme-toggle {
  text-align: right;
  margin-bottom: 10px;
}

.dark .theme-toggle label {
  color: #eee;
}
</style>