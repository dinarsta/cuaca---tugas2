<template>
  <div id="app">
    <div class="weather-card-main">
      <h1 class="title-main">Prakiraan Cuaca Jakarta</h1>

      <!-- Loading -->
      <div v-if="loading" class="loading">Memuat data cuaca...</div>

      <!-- Error -->
      <div v-else-if="error" class="error">Terjadi kesalahan: {{ error }}</div>

      <!-- Isi Data -->
      <div v-else>

        <!-- Card Utama Hari Ini -->
        <div class="main-current-card">
          <div class="city">Jakarta</div>
          <div class="current-temp">{{ todayTemp }}°C</div>

          <div class="row">
            <span>Tertinggi: {{ todayHigh }}°</span>
            <span>Terendah: {{ todayLow }}°</span>
            <span>{{ currentTime }}</span>
          </div>
        </div>

        <!-- Card Per Jam -->
        <div class="section-title-row">
          <h2 class="section-title">Prakiraan Per Jam</h2>

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

        <!-- Card Harian -->
        <h2 class="section-title">Prakiraan 7 Hari</h2>
        <div class="daily-list">
          <div v-for="(item, idx) in dailyForecast" :key="idx" class="day-card">
            <div class="day">{{ item.day }}</div>
            <div class="temps">
              <span>Tertinggi: {{ item.high }}°</span>
              <span>Terendah: {{ item.low }}°</span>
            </div>
          </div>
        </div>

      </div>

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
          "https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&hourly=temperature_2m&daily=temperature_2m_max,temperature_2m_min&timezone=Asia%2FBangkok"
        );

        if (!resp.ok) throw new Error("Gagal mengambil data dari API");

        const data = await resp.json();

        // Per jam
        this.hourlyForecast = data.hourly.time.map((t, i) => ({
          time: t.split('T')[1].slice(0, 5),
          temp: data.hourly.temperature_2m[i],
        }));

        // Per hari
        this.dailyForecast = data.daily.time.map((t, i) => ({
          day: this.formatDay(t, i),
          high: data.daily.temperature_2m_max[i],
          low: data.daily.temperature_2m_min[i],
        }));

        // Summary hari ini
        this.todayTemp = this.hourlyForecast[0]?.temp ?? 26;
        this.todayHigh = data.daily.temperature_2m_max[0] ?? 34;
        this.todayLow = data.daily.temperature_2m_min[0] ?? 24;

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
      const box = this.$refs.hourlyList;
      box.scrollBy({ left: -200, behavior: "smooth" });
    },

    scrollNext() {
      const box = this.$refs.hourlyList;
      box.scrollBy({ left: 200, behavior: "smooth" });
    },

    formatDay(dateStr, idx) {
      const hari = ["Minggu", "Senin", "Selasa", "Rabu", "Kamis", "Jumat", "Sabtu"];
      const d = new Date(dateStr);

      if (idx === 0) return "Hari Ini";
      return hari[d.getDay()] + " (" + d.toLocaleDateString("id-ID") + ")";
    },
  },
};
</script>

<style scoped>
#app {
  width: 100%;
  min-height: 100vh;
  background: linear-gradient(140deg, #82c6ff, #0c74c9);
  display: flex;
  justify-content: center;
  padding: 20px 12px;
  box-sizing: border-box;
  font-family: "Inter", sans-serif;
}

.weather-card-main {
  width: 100%;
  max-width: 540px;
}

/* === MAIN TITLE === */
.title-main {
  color: white;
  font-size: 22px;
  font-weight: 700;
  margin-bottom: 18px;
  text-align: center;
  letter-spacing: 0.3px;
}

/* === CARD UTAMA (Hari Ini) === */
.main-current-card {
  background: rgba(255, 255, 255, 0.9);
  padding: 24px 20px;
  border-radius: 18px;
  margin-bottom: 26px;
  text-align: center;
  border: 1px solid rgba(255, 255, 255, 0.3);
}

.city {
  font-size: 18px;
  font-weight: 600;
  color: #333;
  margin-bottom: 6px;
}

.btn-group {
  display: flex;
  gap: 12px; /* jarak antar tombol */
}

.current-temp {
  font-size: 48px;
  font-weight: 700;
  margin-bottom: 8px;
  color: #0c74c9;
}

/* baris suhu */
.row {
  display: flex;
  justify-content: center;
  gap: 14px;
  color: #555;
  font-size: 14px;
  flex-wrap: wrap;
}

/* === SECTION HEADER === */
.section-title-row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
  margin-top: 4px;
}

.section-title {
  font-size: 16px;
  font-weight: 700;
  color: #fff;
  letter-spacing: 0.3px;
}

.icon-btn {
  border: none;
  outline: none;
}

.icon-btn:focus {
  outline: none;
  box-shadow: none;
  border: none;
}

/* === HOURLY FORECAST === */
.hourly-list {
  display: flex;
  gap: 10px;
  overflow-x: auto;
  scroll-behavior: smooth;
  padding-bottom: 8px;
}

.hourly-list::-webkit-scrollbar {
  height: 5px;
}

.hourly-list::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.3);
  border-radius: 10px;
}

.hour-card {
  min-width: 66px;
  background: rgba(255, 255, 255, 0.92);
  border-radius: 14px;
  padding: 10px 8px;
  text-align: center;
  border: 1px solid rgba(200, 200, 200, 0.25);
}

.hour {
  color: #666;
  font-size: 13px;
  margin-bottom: 4px;
}

.temp {
  font-size: 20px;
  font-weight: 600;
  color: #0c74c9;
}

/* === DAILY FORECAST === */
.daily-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-top: 10px;
}

.day-card {
  background: rgba(255, 255, 255, 0.92);
  padding: 14px 16px;
  border-radius: 14px;
  border: 1px solid rgba(220, 220, 220, 0.4);
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.day {
  font-size: 15px;
  font-weight: 600;
  color: #333;
}

.temps {
  text-align: right;
  font-size: 14px;
  color: #444;
  display: flex;
  flex-direction: column;
  gap: 3px;
}

/* === LOADING & ERROR === */
.loading,
.error {
  text-align: center;
  color: white;
  padding: 20px;
  font-size: 15px;
}

/* === RESPONSIVE === */
@media (max-width: 480px) {
  .current-temp {
    font-size: 42px;
  }

  .main-current-card {
    padding: 20px 16px;
  }

  .hour-card {
    min-width: 60px;
    padding: 8px;
  }

  .day-card {
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }

  .temps {
    text-align: left;
  }
}
</style>
