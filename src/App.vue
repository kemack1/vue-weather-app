<template>
  <main class="app" :style="backgroundStyle">
    <h1>Weather App</h1>

    <SearchBar @search="getWeather" />
    <section class="saved-section">
  <button
    v-if="weather"
    @click="saveLocation"
  >
    Save Current Location
  </button>

  <h2>Saved Locations</h2>

  <p v-if="savedLocations.length === 0">No saved locations yet.</p>

  <ul>
    <li
      v-for="location in savedLocations"
      :key="location"
    >
      <button @click="getWeather(location)">
        {{ location }}
      </button>

      <button
        class="remove-btn"
        @click="removeLocation(location)"
      >
        Remove
      </button>
    </li>
  </ul>
</section>

    <p v-if="loading">Loading...</p>
    <p v-if="error" class="error">{{ error }}</p>

    <section v-if="weather" class="weather-card">
      <h2>{{ weather.location.name }}, {{ weather.location.region }}</h2>

      <img
        :src="'https:' + weather.current.condition.icon"
        :alt="weather.current.condition.text"
      />

      <h3>{{ Math.round(weather.current.temp_f) }}°F</h3>
      <p>{{ weather.current.condition.text }}</p>

      <p>High: {{ Math.round(weather.forecast.forecastday[0].day.maxtemp_f) }}°F</p>
      <p>Low: {{ Math.round(weather.forecast.forecastday[0].day.mintemp_f) }}°F</p>
    </section>
    <section v-if="weather" class="forecast-section">
  <h2>3 Day Forecast</h2>

  <table>
    <thead>
      <tr>
        <th>Date</th>
        <th>Condition</th>
        <th>High</th>
        <th>Low</th>
      </tr>
    </thead>
    <tbody>
      <tr
        v-for="day in weather.forecast.forecastday"
        :key="day.date"
      >
        <td>{{ day.date }}</td>
        <td>
          <img
            :src="'https:' + day.day.condition.icon"
            :alt="day.day.condition.text"
          />
          {{ day.day.condition.text }}
        </td>
        <td>{{ Math.round(day.day.maxtemp_f) }}°F</td>
        <td>{{ Math.round(day.day.mintemp_f) }}°F</td>
      </tr>
    </tbody>
  </table>
<HourlyChart
  v-if="weather"
  :hourlyData="weather.forecast.forecastday[0].hour"
/>
</section>
  </main>
</template>

<script>
import axios from 'axios'
import SearchBar from './components/SearchBar.vue'
import HourlyChart from './components/HourlyChart.vue'

export default {
  components: {
    SearchBar,
    HourlyChart
  },
  data() {
    return {
      weather: null,
      loading: false,
      error: '',
      apiKey: '2894f750d4f14513bf891732262804',
      savedLocations: JSON.parse(localStorage.getItem('savedLocations')) || []
    }
  },
  computed: {
    backgroundStyle() {
      if (!this.weather) return {}

      const condition = this.weather.current.condition.text.toLowerCase()
      const isDay = this.weather.current.is_day

      if (!isDay) {
        return {
          backgroundImage: "url('/src/assets/night.jpg')"
        }
      }

      if (condition.includes('rain')) {
        return {
          backgroundImage: "url('/src/assets/rain.jpg')"
        }
      }

      if (condition.includes('clear') || condition.includes('sunny')) {
        return {
          backgroundImage: "url('/src/assets/clear.jpg')"
        }
      }

      if (this.weather.current.temp_f > 80) {
        return {
          backgroundImage: "url('/src/assets/warm.jpg')"
        }
      }

      return {}
    }
  },
  methods: {
    async getWeather(city) {
      this.loading = true
      this.error = ''
      this.weather = null

      try {
        const response = await axios.get(
          `https://api.weatherapi.com/v1/forecast.json?key=${this.apiKey}&q=${city}&days=3&aqi=no&alerts=no`
        )

        this.weather = response.data
      } catch (err) {
        this.error = 'Could not load weather. Check the city name and try again.'
      } finally {
        this.loading = false
      }
    },
    saveLocation() {
  if (!this.weather) return

  const city = this.weather.location.name

  if (!this.savedLocations.includes(city)) {
    this.savedLocations.push(city)
    localStorage.setItem('savedLocations', JSON.stringify(this.savedLocations))
  }
},

removeLocation(location) {
  this.savedLocations = this.savedLocations.filter(
    savedLocation => savedLocation !== location
  )

  localStorage.setItem('savedLocations', JSON.stringify(this.savedLocations))
}
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Poppins:ital,wght@0,100;0,200;0,300;0,400;0,500;0,600;0,700;0,800;0,900;1,100;1,200;1,300;1,400;1,500;1,600;1,700;1,800;1,900&display=swap');
body {
  margin: 0;
  font-family: 'Poppins', sans-serif;
}
.app {
  padding: 30px;
  min-height: 100vh;
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
}

.app::before {
  content: "";
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.35);
  z-index: -1;
}

h1 {
  text-align: center;
  color: white;
  font-size: 42px;
  margin-bottom: 25px;
}

form {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-bottom: 25px;
}

input {
  padding: 12px 16px;
  width: 280px;
  border: none;
  border-radius: 25px;
  font-size: 16px;
}

button {
  padding: 10px 16px;
  border: none;
  border-radius: 20px;
  background: #2563eb;
  color: white;
  cursor: pointer;
  font-weight: bold;
}

button:hover {
  background: #1d4ed8;
}

.weather-card,
.forecast-section,
.saved-section,
.chart-section {
  background: rgba(255, 255, 255, 0.88);
  backdrop-filter: blur(8px);
  border-radius: 18px;
  padding: 24px;
  margin: 24px auto;
  max-width: 800px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.25);
}

.weather-card {
  text-align: center;
}

.weather-card h2 {
  font-size: 28px;
  margin-bottom: 10px;
}

.weather-card h3 {
  font-size: 56px;
  margin: 10px 0;
}

.weather-card img {
  width: 90px;
}

.saved-section ul {
  padding: 0;
  list-style: none;
}

.saved-section li {
  display: flex;
  justify-content: space-between;
  gap: 10px;
  margin: 10px 0;
}

.remove-btn {
  background: #dc2626;
}

.remove-btn:hover {
  background: #991b1b;
}

table {
  width: 100%;
  border-collapse: collapse;
  overflow: hidden;
  border-radius: 12px;
}

th {
  background: #1f2937;
  color: white;
}

th,
td {
  padding: 12px;
  text-align: center;
  border-bottom: 1px solid #ddd;
}

td img {
  width: 34px;
  vertical-align: middle;
}

.chart-section {
  height: 420px;
}

@media (max-width: 600px) {
  form {
    flex-direction: column;
    align-items: center;
  }

  input {
    width: 90%;
  }

  .weather-card,
  .forecast-section,
  .saved-section,
  .chart-section {
    width: 90%;
    padding: 18px;
  }

  h1 {
    font-size: 32px;
  }

  .weather-card h3 {
    font-size: 44px;
  }
}
</style>
