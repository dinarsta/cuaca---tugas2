<template>
  <div id="app">
    <div class="weather-card-main">
      <h1 class="title-main">Jakarta Weather Forecast</h1>

      <!-- Loading -->
      <div v-if="loading" class="loading">Memuat data cuaca...</div>

      <!-- Error -->
      <div v-else-if="error" class="error">Terjadi kesalahan: {{ error }}</div>

      <!-- Isi Data -->
      <div v-else>

        <!-- Card Utama -->
        <div class="main-current-card">
          <div class="city">Jakarta</div>
          <div class="current-temp">{{ todayTemp }}°C</div>

          <div class="row">
            <span>Tertinggi: {{ todayHigh }}°</span>
            <span>Terendah: {{ todayLow }}°</span>
            <span>{{ currentTime }}</span>
          </div>

          <div class="extra-info">
            <div class="info-box">
              <i class="fas fa-tint"></i> {{ humidity }}%
              <span>Kelembapan</span>
            </div>

            <div class="info-box">
              <i class="fas fa-wind"></i> {{ wind }} km/h
              <span>Angin</span>
            </div>
          </div>
        </div>

        <!-- Per Jam -->
        <div class="section-title-row">
          <h2 class="section-title">Hourly Weather Update</h2>

          <div class="btn-group">
            <button class="icon-btn" @click="scrollPrev">←</button>
            <button class="icon-btn" @click="scrollNext">→</button>
          </div>
        </div>

        <div class="hourly-list" ref="hourlyList">
          <div v-for="(item, idx) in hourlyForecast" :key="idx" class="hour-card">
            <div class="hour">{{ item.time }}</div>
            <div class="temp">{{ item.temp }}°</div>
          </div>
        </div>

        <!-- Harian -->
        <h2 class="section-title">Daily Weather</h2>

        <div class="calendar-grid">
          <div v-for="(item, idx) in dailyForecast" :key="idx" class="calendar-day">
            <div class="day-name">{{ item.dayName }}</div>
            <div class="day-date">{{ item.date }}</div>
            <div class="day-temps">
              <span>↑ {{ item.high }}°</span>
              <span>↓ {{ item.low }}°</span>
            </div>
          </div>
        </div>

      </div>

      <footer class="footer">Made by Dina Rosita</footer>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      loading: true,
      error: null,

      todayTemp: 26,
      todayHigh: 34,
      todayLow: 24,
      currentTime: "",

      humidity: 0,
      wind: 0,

      hourlyForecast: [],
      dailyForecast: [],
    };
  },

  mounted() {
    this.fetchWeatherData();
  },

  methods: {
    async fetchWeatherData() {
      try {
        const resp = await fetch(
          "https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&hourly=temperature_2m,relativehumidity_2m,windspeed_10m&daily=temperature_2m_max,temperature_2m_min&days=14&timezone=Asia%2FBangkok"
        );

        if (!resp.ok) throw new Error("Gagal mengambil data dari API");

        const data = await resp.json();

        this.hourlyForecast = data.hourly.time.map((t, i) => ({
          time: t.split('T')[1].slice(0, 5),
          temp: data.hourly.temperature_2m[i],
        }));

        this.dailyForecast = data.daily.time.map((t, i) => {
          const d = new Date(t);
          return {
            dayName: this.formatDayName(d, i),
            date: d.toLocaleDateString("id-ID", {
              day: "numeric",
              month: "short",
            }),
            high: data.daily.temperature_2m_max[i],
            low: data.daily.temperature_2m_min[i],
          };
        });

        this.todayTemp = this.hourlyForecast[0]?.temp ?? 26;
        this.todayHigh = data.daily.temperature_2m_max[0];
        this.todayLow = data.daily.temperature_2m_min[0];

        this.humidity = data.hourly.relativehumidity_2m[0] ?? 0;
        this.wind = data.hourly.windspeed_10m[0] ?? 0;

        this.currentTime =
          new Date().toLocaleTimeString("id-ID", {
            hour: "2-digit",
            minute: "2-digit",
          }) + " WIB";

        this.loading = false;
      } catch (e) {
        this.error = e.message;
        this.loading = false;
      }
    },

    scrollPrev() {
      this.$refs.hourlyList.scrollBy({ left: -200, behavior: "smooth" });
    },

    scrollNext() {
      this.$refs.hourlyList.scrollBy({ left: 200, behavior: "smooth" });
    },

    formatDayName(date, idx) {
      const hari = ["Minggu", "Senin", "Selasa", "Rabu", "Kamis", "Jumat", "Sabtu"];
      return idx === 0 ? "Hari Ini" : hari[date.getDay()];
    },
  },
};
</script>

<style src="./assets/weather.css"></style>
