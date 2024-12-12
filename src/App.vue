<template>
  <div id="app">
    <div class="container">
      <h1 class="title">Weather Forecast Application</h1>
      <div class="search-container">
        <input
          type="text"
          v-model="query"
          placeholder="Enter a city name"
          @keyup.enter="searchLocation"
          class="search-input"
        />
        <button @click="searchLocation" class="search-button">Search</button>
      </div>

      <div v-if="error" class="error">{{ error }}</div>
      <div v-if="loading" class="loading">Loading...</div>

      <div v-if="weatherData" class="weather-container">
        <h2 class="weather-title">Weather in {{ weatherData.city }}</h2>
        <p>Current Temperature: <strong>{{ weatherData.currentTemperature }}°C</strong></p>
        <ul>
          <li v-for="(day, index) in weatherData.dailyForecast" :key="index">
            <span>{{ day.date }}</span>: <strong>{{ day.temperature }}°C</strong>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      query: "",
      loading: false,
      error: null,
      weatherData: null,
    };
  },
  methods: {
    async searchLocation() {
      this.error = null;
      this.weatherData = null;

      if (!this.query) {
        this.error = "Please enter a city name.";
        return;
      }

      this.loading = true;

      try {
        const locationResponse = await fetch(
          `https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(
            this.query
          )}&count=1&language=en&format=json`
        );
        const locationData = await locationResponse.json();

        if (!locationData.results || locationData.results.length === 0) {
          this.error = "City not found.";
          return;
        }

        const { latitude, longitude, name } = locationData.results[0];

        const weatherResponse = await fetch(
          `https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&daily=temperature_2m_max,temperature_2m_min&timezone=auto`
        );
        const weatherData = await weatherResponse.json();

        this.weatherData = {
          city: name,
          currentTemperature: weatherData.daily.temperature_2m_max[0],
          dailyForecast: weatherData.daily.time.map((date, index) => ({
            date,
            temperature: weatherData.daily.temperature_2m_max[index],
          })),
        };
      } catch {
        this.error = "Failed to fetch data. Please try again.";
      } finally {
        this.loading = false;
      }
    },
  },
};
</script>
<style src="./AppStyles.css"></style>