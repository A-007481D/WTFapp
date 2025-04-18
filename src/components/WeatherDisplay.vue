<template>
  <div>
    <div class="text-center mb-4">
      <h2 class="text-xl font-semibold">{{ weather.name }}, {{ weather.sys.country }}</h2>
      <div class="flex justify-center items-center">
        <img
            :src="`https://openweathermap.org/img/wn/${weather.weather[0].icon}@2x.png`"
            :alt="weather.weather[0].description"
            class="w-16 h-16"
        />
        <div class="text-4xl font-bold">{{ Math.round(weather.main.temp) }}°C</div>
      </div>
      <p class="capitalize">{{ weather.weather[0].description }}</p>
    </div>

    <div class="grid grid-cols-2 gap-2 mb-4">
      <div class="bg-gray-100 p-2 rounded">
        <p class="text-sm text-gray-500">Feels like</p>
        <p class="font-medium">{{ Math.round(weather.main.feels_like) }}°C</p>
      </div>
      <div class="bg-gray-100 p-2 rounded">
        <p class="text-sm text-gray-500">Humidity</p>
        <p class="font-medium">{{ weather.main.humidity }}%</p>
      </div>
      <div class="bg-gray-100 p-2 rounded">
        <p class="text-sm text-gray-500">Wind</p>
        <p class="font-medium">{{ Math.round(weather.wind.speed * 3.6) }} km/h</p>
      </div>
      <div class="bg-gray-100 p-2 rounded">
        <p class="text-sm text-gray-500">Pressure</p>
        <p class="font-medium">{{ weather.main.pressure }} hPa</p>
      </div>
    </div>

    <div v-if="forecast && forecast.list">
      <h3 class="font-semibold mb-2">Forecast</h3>
      <div class="flex overflow-x-auto pb-2">
        <div
            v-for="(item, index) in forecast.list.slice(0, 8)"
            :key="index"
            class="flex-shrink-0 w-20 text-center mr-2"
        >
          <p class="text-xs">{{ formatTime(item.dt) }}</p>
          <img
              :src="`https://openweathermap.org/img/wn/${item.weather[0].icon}.png`"
              :alt="item.weather[0].description"
              class="w-10 h-10 mx-auto"
          />
          <p class="font-medium">{{ Math.round(item.main.temp) }}°C</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'WeatherDisplay',
  props: {
    weather: {
      type: Object,
      required: true
    },
    forecast: {
      type: Object,
      default: null
    }
  },
  methods: {
    formatTime(timestamp) {
      const date = new Date(timestamp * 1000);
      return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
    }
  }
};
</script>