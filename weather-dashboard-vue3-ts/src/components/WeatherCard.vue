<template>
  <div class="weather-card card">
    <div style="display:flex;justify-content:space-between;align-items:center">
      <div>
        <div class="big">{{ data.name }}</div>
        <div class="muted small">{{ data.weather[0].description }}</div>
      </div>
      <div style="text-align:right">
        <div class="big">{{ Math.round(data.main.temp) }}°C</div>
        <div class="muted small">Wind {{ data.wind.speed }} m/s</div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType } from 'vue'

interface WeatherAPIResponse {
  name: string
  main: { temp: number; humidity: number }
  weather: { main: string; description: string; icon: string }[]
  wind: { speed: number }
}

export default defineComponent({
  name: 'WeatherCard',
  props: {
    data: { type: Object as PropType<WeatherAPIResponse>, required: true }
  }
})
</script>
