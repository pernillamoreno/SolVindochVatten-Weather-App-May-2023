<template>
  <div id="app">
    <header>
      <h1>SolVindochVatten Weather App</h1>
    </header>

    <div class="recent-locations">
      <section v-if="savedLocations.length !== 0" class="searched-locations">
        <div
          v-for="location in savedLocations"
          :key="location"
          class="city-name"
        >
          <button @click="getWeather(location)" class="location-button">
            {{ location }}
          </button>
        </div>
      </section>
    </div>

    <div class="search-field">
      <input type="text" v-model="searchQuery" placeholder="Enter location" />
      <button @click="searchWeather">Click here!</button>
    </div>

    <div class="weather-info" v-if="lon !== 0 || lat !== 0">
      <h3>{{ weatherDetails.location }}</h3>
      <section class="temperature-and-description">
        <p>{{ weatherDetails.temperature }}°C</p>
        <p>{{ weatherDetails.description }}</p>
        <img :src="weatherDetails.icon" alt="weather-icon" />
      </section>

      <section class="weather-info-details">
        <p>Humidity: {{ weatherDetails.humidity }}%</p>
        <p>Wind Speed: {{ weatherDetails.wind }} m/s</p>
        <p>Date and Time: {{ getFormattedDateTime() }}</p>
      </section>
    </div>

    <div class="weather-forecast-container">
      <h2 class="weather-forecast">Weather forecast</h2>
      <ul
        v-for="weatherDetails in weatherForecast"
        :key="weatherDetails.time"
        class="weather-forecast-info"
      >
        <img :src="weatherDetails.icon" />
        <p class="weather-forecast-detail">
          {{ weatherDetails.temperature }}°C
          {{ getFormattedTime(weatherDetails.time) }}
        </p>
      </ul>
    </div>
  </div>
</template>
<script lang="ts">
import axios, { type AxiosResponse } from "axios";
import { defineComponent } from "vue";
import { type WeatherDetails } from "../model/weatherDetails";

export default defineComponent({
  name: "SolVindOchVatten Weather App",
  data() {
    return {
      savedLocations: [] as string[],
      searchQuery: "",
      location: "",
      lon: null,
      lat: null,
      weatherDetails: {} as WeatherDetails,
      weatherForecast: [] as WeatherDetails[],
    };
  },

  mounted() {
    this.savedLocations = JSON.parse(
      localStorage.getItem("savedLocations") || "[]"
    );
  },

  methods: {
    getWeather(location: string): void {
      this.fetchWeatherData(location);
      this.storeLocation(location);
    },

    searchWeather(): void {
      if (this.searchQuery.trim() === "") {
        return;
      }
      this.fetchWeatherData(this.searchQuery);
      this.storeLocation(this.searchQuery);
      this.searchQuery = "";
    },

    storeLocation(location: string): void {
      if (this.savedLocations.length === 3) {
        this.savedLocations.pop();
      }
      this.savedLocations.unshift(location);
      localStorage.setItem(
        "savedLocations",
        JSON.stringify(this.savedLocations)
      );
    },

    resetSearch() {
      this.searchQuery = "";
    },

    getWeatherIconURL(icon: string): string {
      const apiKey = "619d1caa21f25da4216719cb6d4dcc47";
      return `https://openweathermap.org/img/w/${icon}.png?appid=${apiKey}`;
    },

    getFormattedDateTime(): string {
      const date = new Date();
      const options: Intl.DateTimeFormatOptions = {
        weekday: "long",
        year: "numeric",
        month: "long",
        day: "numeric",
        hour: "numeric",
        minute: "numeric",
        second: "numeric",
        timeZoneName: "short",
      };
      return date.toLocaleString("en-US", options);
    },

    getFormattedTime(dt: number): string {
      const date = new Date(dt * 2000);
      return date.toLocaleTimeString(navigator.language, {
        hour: "2-digit",
        minute: "2-digit",
      });
    },

    async fetchWeatherForecast(location: string): Promise<void> {
      const apiKey = "619d1caa21f25da4216719cb6d4dcc47";
      const forecastUrl = `https://api.openweathermap.org/data/2.5/forecast?q=${location}&cnt=3&appid=${apiKey}&units=metric`;
      try {
        const response: AxiosResponse = await axios.get(forecastUrl);
        this.weatherForecast = response.data.list.map((forecast: any) => ({
          temperature: Math.round(forecast.main.temp),
          time: forecast.dt,
          icon: this.getWeatherIconURL(forecast.weather[0].icon),
        }));
      } catch (error) {
        console.error("Error when fetching weather forecast: ", error);
      }
    },
    async fetchWeatherData(location: string): Promise<void> {
      const apiKey = "619d1caa21f25da4216719cb6d4dcc47";
      const url = `https://api.openweathermap.org/data/2.5/weather?q=${location}&appid=${apiKey}&units=metric`;
      try {
        const response: AxiosResponse = await axios.get(url);
        const temperature = Math.round(response.data.main.temp);
        this.weatherDetails = {
          location: response.data.location,
          temperature: temperature,
          description: response.data.weather[0].description,
          time: Date.now(),
          humidity: response.data.main.humidity,
          wind: response.data.wind,
          icon: this.getWeatherIconURL(response.data.weather[0].icon),
        };
        await this.fetchWeatherForecast(location);
      } catch (error) {
        console.error("Error when fetching weather data: ", error);
      }
    },
  },
});
</script>

<style scoped>
header {
  font-family: "Playful";
  font-size: 18px;
  background-image: url(sol.jpg);
  text-align: center;
  padding: 2em;
  margin-bottom: 1em;
}
.search-field {
  font-family: "Playful";
  margin-bottom: 0.5em;
  width: 100%;
  height: 30px;
  font-size: 24px;
  padding: 0.5em.25em;
  text-align: center;
}

.app {
  font-family: "Playful";
}
.searched-locations {
  font-family: "Playful";
  display: flex;
  justify-content: center;
  gap: 0.5em;
}
.city-name {
  font-family: "Playful";
  padding: 0em 1em;
  border-radius: 7px;
  margin-bottom: 1.5em;
}
.location-button {
  background-color: #7fceff;
  font-family: "Playful";
  font-size: 20px;
  padding: 0.5em 1em;
}

p {
  font-family: "hand-crafted";
  font-size: 65%;
}

.weather-info {
  background-color: #7fceff;
  font-family: "Playful";
  padding: 1em 2em 2em 2em;
  margin-bottom: 1.5em;
  border-style: dotted;
}
.weather-info > h3 {
  font-size: 30px;
}

.location-name {
  margin-bottom: 1.5em;
  font-size: 25px;
}
.temperature-and-description {
  font-family: "Playful";
  display: flex;
  justify-content: center;
  justify-content: space-between;
  margin-bottom: 5em;
  font-size: 25px;
}
.weather-info-details {
  font-family: "Playful";
  justify-content: center;
  margin-bottom: 2em;
  font-size: 25px;
  display: flex;
  gap: 5em;
}

.weather-forecast {
  font-family: "Playful";
  text-align: center;
  margin-bottom: 0.25em;
  font-size: 25px;
}
.weather-forecast-container {
  padding: 2em;
  border-style: dotted;
  margin-bottom: 1.5em;
}
.weather-forecast-info {
  display: flex;
  justify-content: center;
  gap: 30px;
}
.weather-forecast-detail {
  font-size: 15px;
}

.location-name {
  font-family: "Playful";
  text-align: center;
  margin-bottom: 0.25em;
  font-size: 25px;
}
</style>
