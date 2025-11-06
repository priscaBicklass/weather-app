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

      <div v-if="!weather && !loading" class="muted" style="margin-top: 12px">
        Try searching for a city — e.g. "Lilongwe", "Tokyo", or "London".
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { ref } from "vue";
import WeatherCard from "./components/WeatherCard.vue";

interface WeatherAPIResponse {
  name: string;
  main: { temp: number; humidity: number };
  weather: { main: string; description: string; icon: string }[];
  wind: { speed: number };
}

const API_KEY = import.meta.env.VITE_OPENWEATHER_API_KEY;

export default {
  components: { WeatherCard },
  setup() {
    const query = ref("");
    const weather = ref<WeatherAPIResponse | null>(null);
    const loading = ref(false);
    const error = ref<string | null>(null);

    async function search() {
      if (!query.value) return;
      loading.value = true;
      error.value = null;
      weather.value = null;
      try {
        const q = encodeURIComponent(query.value.trim());
        const res = await fetch(
          `https://api.openweathermap.org/data/2.5/weather?q=${q}&units=metric&appid=${API_KEY}`
        );
        if (!res.ok) {
          const body = await res.json().catch(() => ({}));
          throw new Error(body.message || "Failed to fetch");
        }
        const data = (await res.json()) as WeatherAPIResponse;
        weather.value = data;
      } catch (err: any) {
        error.value = err.message || String(err);
      } finally {
        loading.value = false;
      }
    }

    return { query, weather, loading, error, search };
  },
};
</script>
