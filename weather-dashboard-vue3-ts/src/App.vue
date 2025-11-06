<template>
  <div class="container">
    <h1>Weather Dashboard</h1>

    <!-- 🔍 Search input -->
    <div class="search-row">
      <input
        v-model="query"
        @keyup.enter="search"
        placeholder="Enter city name..."
        class="input"
      />
      <button @click="search" class="btn">Search</button>
    </div>

    <!-- 🕓 Recent searches -->
    <div class="history" v-if="history.length">
      <h3>Recent Searches</h3>
      <div class="history-buttons">
        <button
          v-for="(city, index) in history"
          :key="index"
          @click="search(city)"
          class="history-btn"
        >
          {{ city }}
        </button>
      </div>
    </div>

    <!-- 🌤️ Current weather -->
    <div class="grid" v-if="weather">
      <WeatherCard :data="weather" />
    </div>

    <!-- 📅 5-day forecast -->
    <div class="forecast" v-if="forecast.length">
      <h3>5-Day Forecast</h3>
      <div class="forecast-grid">
        <WeatherCard
          v-for="(day, index) in forecast"
          :key="index"
          :data="day"
        />
      </div>
    </div>

    <!-- ⚠️ Error message -->
    <p v-if="error" class="error">{{ error }}</p>

    <!-- ⏳ Loading indicator -->
    <p v-if="loading" class="loading">Loading...</p>
  </div>
</template>

<script lang="ts">
import { ref } from "vue";
import WeatherCard from "./components/WeatherCard.vue";

const API_KEY = "fe8196c431caa4c419fe319a2f7816da";

export default {
  name: "App",
  components: { WeatherCard },
  setup() {
    const query = ref("");
    const weather = ref<any>(null);
    const forecast = ref<any[]>([]);
    const loading = ref(false);
    const error = ref<string | null>(null);
    const history = ref<string[]>(
      JSON.parse(localStorage.getItem("searchHistory") || "[]")
    );

    // ✅ Search function (accepts string or event)
    async function search(arg?: string | KeyboardEvent | PointerEvent) {
      const searchQuery = typeof arg === "string" ? arg : query.value.trim();
      if (!searchQuery) return;

      loading.value = true;
      error.value = null;
      weather.value = null;
      forecast.value = [];

      try {
        // 🟦 Current weather
        const currentRes = await fetch(
          `https://api.openweathermap.org/data/2.5/weather?q=${encodeURIComponent(
            searchQuery
          )}&units=metric&appid=${API_KEY}`
        );
        if (!currentRes.ok) throw new Error("City not found");
        weather.value = await currentRes.json();

        // 🟩 5-day forecast
        const forecastRes = await fetch(
          `https://api.openweathermap.org/data/2.5/forecast?q=${encodeURIComponent(
            searchQuery
          )}&units=metric&appid=${API_KEY}`
        );
        if (!forecastRes.ok) throw new Error("Unable to fetch forecast");
        const data = await forecastRes.json();
        forecast.value = data.list.filter((_: any, i: number) => i % 8 === 0);

        // 💾 Update search history
        if (!history.value.includes(searchQuery)) {
          history.value.unshift(searchQuery);
          if (history.value.length > 5) history.value.pop();
          localStorage.setItem("searchHistory", JSON.stringify(history.value));
        }
      } catch (err: any) {
        error.value = err.message || String(err);
      } finally {
        loading.value = false;
      }
    }

    return { query, weather, forecast, loading, error, history, search };
  },
};
</script>

<style scoped>
.container {
  max-width: 700px;
  margin: 2rem auto;
  text-align: center;
  font-family: Arial, sans-serif;
}

.search-row {
  display: flex;
  justify-content: center;
  gap: 0.5rem;
}

.input {
  padding: 0.6rem;
  width: 60%;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.btn {
  padding: 0.6rem 1rem;
  border: none;
  background-color: #3b82f6;
  color: white;
  border-radius: 4px;
  cursor: pointer;
}
.btn:hover {
  background-color: #2563eb;
}

.history {
  margin-top: 1rem;
}
.history-buttons {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 0.4rem;
}
.history-btn {
  background: #e5e7eb;
  border: none;
  border-radius: 4px;
  padding: 0.4rem 0.8rem;
  cursor: pointer;
}
.history-btn:hover {
  background: #d1d5db;
}

.grid,
.forecast-grid {
  display: grid;
  gap: 1rem;
  justify-content: center;
}

.forecast-grid {
  grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
}

.error {
  color: red;
  margin-top: 1rem;
}

.loading {
  color: #555;
  margin-top: 1rem;
}
</style>
