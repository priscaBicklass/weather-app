<template>
  <div class="app">
    <h1 class="title">🌤️ Weather Dashboard</h1>

    <!-- Search Section -->
    <div class="search-row">
      <input
        v-model="query"
        @keyup.enter="search"
        placeholder="Enter city name..."
        class="input"
      />
      <button @click="search" class="btn">Search</button>
    </div>

    <!-- Search History -->
    <div v-if="history.length" class="history">
      <h3>Recent Searches:</h3>
      <div class="history-buttons">
        <button
          v-for="city in history"
          :key="city"
          class="history-btn"
          @click="search(city)"
        >
          {{ city }}
        </button>
      </div>
    </div>

    <!-- Messages -->
    <p v-if="error" class="error">{{ error }}</p>
    <p v-if="loading" class="loading">Loading...</p>

    <!-- Weather Content -->
    <div v-if="weather || forecast.length" class="dashboard">
      <!-- Current Weather -->
      <div v-if="weather" class="current-weather">
        <h2>{{ weather.name }}</h2>
        <div class="weather-main">
          <img
            :src="`https://openweathermap.org/img/wn/${weather.weather[0].icon}@2x.png`"
            :alt="weather.weather[0].description"
            class="weather-icon"
          />
          <div class="weather-info">
            <p class="temp">{{ Math.round(weather.main.temp) }}°C</p>
            <p class="weather-desc">{{ weather.weather[0].main }}</p>
            <p class="details">
              Humidity: {{ weather.main.humidity }}% | Wind:
              {{ weather.wind.speed }} m/s
            </p>
          </div>
        </div>
      </div>

      <!-- 5-Day Forecast -->
      <div v-if="forecast.length" class="forecast">
        <h3>5-Day Forecast</h3>
        <div class="forecast-grid">
          <div v-for="day in forecast" :key="day.dt" class="forecast-card">
            <p class="day">{{ formatDay(day.dt) }}</p>
            <img
              :src="`https://openweathermap.org/img/wn/${day.weather[0].icon}.png`"
              :alt="day.weather[0].description"
              class="forecast-icon"
            />
            <p class="temp">{{ Math.round(day.main.temp) }}°C</p>
            <p class="desc">{{ day.weather[0].main }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { ref } from "vue";

export default {
  setup() {
    const API_KEY = "fe8196c431caa4c419fe319a2f7816da";
    const query = ref("");
    const weather = ref<any>(null);
    const forecast = ref<any[]>([]);
    const loading = ref(false);
    const error = ref<string | null>(null);
    const history = ref<string[]>(
      JSON.parse(localStorage.getItem("history") || "[]")
    );

    function formatDay(timestamp: number) {
      const date = new Date(timestamp * 1000);
      return date.toLocaleDateString("en-US", { weekday: "short" });
    }

    async function search(arg?: string | KeyboardEvent | PointerEvent) {
      const searchQuery = typeof arg === "string" ? arg : query.value.trim();
      if (!searchQuery) return;

      loading.value = true;
      error.value = null;
      weather.value = null;
      forecast.value = [];

      try {
        // Current weather
        const currentRes = await fetch(
          `https://api.openweathermap.org/data/2.5/weather?q=${encodeURIComponent(
            searchQuery
          )}&units=metric&appid=${API_KEY}`
        );
        if (!currentRes.ok) throw new Error("City not found");
        weather.value = await currentRes.json();

        // Forecast
        const forecastRes = await fetch(
          `https://api.openweathermap.org/data/2.5/forecast?q=${encodeURIComponent(
            searchQuery
          )}&units=metric&appid=${API_KEY}`
        );
        if (!forecastRes.ok) throw new Error("Unable to fetch forecast");
        const data = await forecastRes.json();

        // Pick one data point per day (every 8th)
        forecast.value = data.list.filter(
          (_: any, index: number) => index % 8 === 0
        );

        // Save search history
        if (!history.value.includes(searchQuery)) {
          history.value.unshift(searchQuery);
          if (history.value.length > 5) history.value.pop();
          localStorage.setItem("history", JSON.stringify(history.value));
        }
      } catch (err: any) {
        error.value = err.message || String(err);
      } finally {
        loading.value = false;
      }
    }

    return {
      query,
      weather,
      forecast,
      loading,
      error,
      history,
      search,
      formatDay,
    };
  },
};
</script>

<style scoped>
.app {
  background-color: #121212;
  color: #f0f0f0;
  min-height: 100vh;
  padding: 1rem;
  font-family: "Inter", sans-serif;
  width: 100%;
}

.title {
  text-align: center;
  font-size: 1.5rem;
  margin-bottom: 1rem;
  color: #4fc3f7;
}

.search-row {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  margin-bottom: 1rem;
  width: 100%;
}

.input {
  padding: 0.75rem 1rem;
  border-radius: 0.5rem;
  border: none;
  background: #1e1e1e;
  color: white;
  font-size: 1rem;
  width: 100%;
}

.btn {
  background: #4fc3f7;
  color: #000;
  border: none;
  border-radius: 0.5rem;
  padding: 0.75rem;
  cursor: pointer;
  font-size: 1rem;
  font-weight: 600;
  width: 100%;
}

/* History */
.history {
  text-align: center;
  margin-bottom: 1.5rem;
}

.history-buttons {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin-top: 0.5rem;
}

.history-btn {
  background: #2a2a2a;
  color: #fff;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 0.4rem;
  cursor: pointer;
  font-size: 0.9rem;
}

/* Messages */
.error {
  color: #ff5252;
  text-align: center;
  background: rgba(255, 82, 82, 0.1);
  padding: 1rem;
  border-radius: 0.5rem;
  margin: 1rem 0;
}

.loading {
  color: #4fc3f7;
  text-align: center;
  padding: 1rem;
}

.dashboard {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  width: 100%;
}

/* Current Weather */
.current-weather {
  background: #1c1c1c;
  border-radius: 1rem;
  padding: 1.5rem;
  box-shadow: 0 0 15px rgba(79, 195, 247, 0.2);
  width: 100%;
}

.weather-main {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  text-align: center;
}

.weather-icon {
  width: 80px;
  height: 80px;
}

.temp {
  font-size: 2rem;
  font-weight: bold;
  margin-bottom: 0.5rem;
}

.weather-desc {
  font-size: 1.2rem;
  margin-bottom: 0.5rem;
  text-transform: capitalize;
}

.details {
  font-size: 0.9rem;
  opacity: 0.8;
}

/* Forecast */
.forecast h3 {
  text-align: center;
  margin-bottom: 1rem;
  font-size: 1.3rem;
}

.forecast-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 0.75rem;
}

.forecast-card {
  background: #1c1c1c;
  border-radius: 0.75rem;
  padding: 1rem;
  text-align: center;
  box-shadow: 0 0 10px rgba(79, 195, 247, 0.15);
}

.forecast-icon {
  width: 50px;
  height: 50px;
  margin: 0.5rem auto;
}

.day {
  font-weight: bold;
  color: #4fc3f7;
}

@media (min-width: 768px) {
  .app {
    padding: 2rem;
    max-width: 1200px;
    margin: 0 auto;
  }

  .title {
    font-size: 2rem;
  }

  .search-row {
    flex-direction: row;
    justify-content: center;
  }

  .input {
    width: 300px;
  }

  .btn {
    width: auto;
    padding: 0.75rem 1.5rem;
  }

  .weather-main {
    flex-direction: row;
    text-align: left;
    justify-content: center;
    gap: 2rem;
  }

  .forecast-grid {
    grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
    gap: 1rem;
  }
}

@media (min-width: 1024px) {
  .dashboard {
    flex-direction: row;
    align-items: flex-start;
  }

  .current-weather {
    flex: 1;
    max-width: 400px;
  }

  .forecast {
    flex: 2;
  }
}

@media (min-width: 1200px) {
  .forecast-grid {
    grid-template-columns: repeat(5, 1fr);
  }
}
</style>
