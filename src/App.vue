<template>
  <div :class="darkMode ? 'dark bg-gray-900 text-white' : 'bg-gray-100 text-gray-800'" class="min-h-screen transition-colors">
    <div class="max-w-2xl mx-auto p-6">

      <!-- Header -->
      <div class="flex items-center justify-between mb-6">
        <h1 class="text-3xl font-bold">Jakarta Weather</h1>

        <!-- Dark mode toggle -->
        <button
          @click="darkMode = !darkMode"
          class="px-3 py-1 rounded-xl border dark:border-white border-gray-700"
        >
          {{ darkMode ? 'Light' : 'Dark' }}
        </button>
      </div>

      <!-- Weather card -->
      <div class="p-6 rounded-xl shadow bg-white dark:bg-gray-800 dark:text-white">
        <div class="flex items-center gap-4">
          <div class="text-5xl">
            {{ weatherIcon }}
          </div>

          <div>
            <p class="text-lg font-semibold">Current Temperature</p>
            <p class="text-2xl font-bold">{{ currentTemp }}°C</p>
          </div>
        </div>
      </div>

      <!-- Table -->
      <h2 class="mt-8 mb-3 text-xl font-semibold">Hourly Forecast</h2>

      <div class="overflow-auto rounded-xl shadow">
        <table class="min-w-full text-left bg-white dark:bg-gray-800 dark:text-white">
          <thead class="bg-gray-200 dark:bg-gray-700">
            <tr>
              <th class="p-3">Time</th>
              <th class="p-3">Temperature (°C)</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(t, i) in data.time" :key="i" class="border-b dark:border-gray-700">
              <td class="p-3">{{ t }}</td>
              <td class="p-3">{{ data.temperature_2m[i] }}</td>
            </tr>
          </tbody>
        </table>
      </div>

    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'

const data = ref({ time: [], temperature_2m: [], weathercode: [] })
const darkMode = ref(false)

const currentTemp = computed(() =>
  data.value.temperature_2m.length ? data.value.temperature_2m[0] : '--'
)

const weatherIcon = computed(() => {
  const code = data.value.weathercode[0]
  if (code === undefined) return '⛅'

  if (code === 0) return '☀️'      // clear
  if ([1,2,3].includes(code)) return '⛅' // partly cloudy
  if ([45,48].includes(code)) return '🌫️' // fog
  if ([51,53,55].includes(code)) return '🌦️' // drizzle
  if ([61,63,65].includes(code)) return '🌧️' // rain
  if ([71,73,75].includes(code)) return '❄️' // snow
  if ([95,96,99].includes(code)) return '⛈️' // thunderstorm
  return '🌡️'
})

onMounted(async () => {
  const res = await fetch(
    'https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&hourly=temperature_2m,weathercode'
  )
  const json = await res.json()
  data.value = json.hourly
})
</script>

<style>
/* optional smooth transition */
html, body {
  transition: background-color .3s ease, color .3s ease;
}
</style>
