<template>
  <div class="container">
    <div class="card">
      <div class="header">
        <div>
          <h2>Weather Dashboard</h2>
          <p class="muted">Search a city to see current weather</p>
        </div>
        <div>
          <input
            v-model="query"
            @keyup.enter="search"
            placeholder="e.g. Tokyo"
          />
          <button @click="search">Search</button>
        </div>
      </div>

      <div v-if="error" class="muted">Error: {{ error }}</div>

      <div v-if="loading" class="muted">Loading…</div>

      <div class="grid" v-if="weather">
        <WeatherCard :data="weather" />
      </div>
      <div v-if="forecast.length > 0" class="forecast-section">
        <h3>5-Day Forecast</h3>
        <div class="forecast-grid">
          <div v-for="day in forecast" :key="day.dt" class="forecast-card">
            <p>
              {{
                day.dt
                  ? new Date(day.dt * 1000).toLocaleDateString(undefined, {
                      weekday: "short",
                    })
                  : "N/A"
              }}
            </p>
            <p>{{ Math.round(day.main.temp) }}°C</p>
            <p>{{ day.weather[0].main }}</p>
          </div>
        </div>
      </div>

      <div v-if="!weather && !loading" class="muted" style="margin-top: 12px">
        Try searching for a city — e.g. "Lilongwe", "Tokyo", or "London".
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { ref } from "vue";
interface WeatherAPIResponse {
  name?: string;
  main: { temp: number; humidity: number };
  weather: { main: string; description: string; icon: string }[];
  wind: { speed: number };
  dt?: number;
}

const API_KEY = import.meta.env.VITE_OPENWEATHER_API_KEY;

export default {
  setup() {
    const query = ref("");
    const weather = ref<WeatherAPIResponse | null>(null);
    const forecast = ref<WeatherAPIResponse[]>([]);
    const loading = ref(false);
    const error = ref<string | null>(null);

    async function search() {
      if (!query.value.trim()) return;
      loading.value = true;
      error.value = null;
      weather.value = null;
      forecast.value = [];

      try {
        // ✅ Current weather
        const currentRes = await fetch(
          `https://api.openweathermap.org/data/2.5/weather?q=${encodeURIComponent(
            query.value
          )}&units=metric&appid=${API_KEY}`
        );
        if (!currentRes.ok) throw new Error("City not found");
        weather.value = await currentRes.json();

        // ✅ 5-day forecast (every 3 hours)
        const forecastRes = await fetch(
          `https://api.openweathermap.org/data/2.5/forecast?q=${encodeURIComponent(
            query.value
          )}&units=metric&appid=${API_KEY}`
        );
        if (!forecastRes.ok) throw new Error("Unable to fetch forecast");
        const data = await forecastRes.json();

        // ✅ Pick one data point per day (every 8th item)
        forecast.value = data.list.filter(
          (_: any, index: number) => index % 8 === 0
        );
      } catch (err: any) {
        error.value = err.message || String(err);
      } finally {
        loading.value = false;
      }
    }

    return { query, weather, forecast, loading, error, search };
  },
};
</script>
