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

        <!-- Card Utama Hari Ini -->
        <div class="main-current-card">
          <div class="city">Jakarta</div>
          <div class="current-temp">{{ todayTemp }}°C</div>

          <div class="row">
            <span>Tertinggi: {{ todayHigh }}°</span>
            <span>Terendah: {{ todayLow }}°</span>
            <span>{{ currentTime }}</span>
          </div>

          <!-- DETAIL TAMBAHAN -->
          <div class="extra-info">
            <div class="info-box">
              💧 {{ humidity }}%
              <span>Kelembapan</span>
            </div>

            <div class="info-box">
              🌬️ {{ wind }} km/h
              <span>Angin</span>
            </div>
          </div>
        </div>

        <!-- Card Per Jam -->
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

        <!-- Card Harian (Calendar-like Grid) -->
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

      <!-- Footer -->
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

      // cuaca hari ini
      todayTemp: 26,
      todayHigh: 34,
      todayLow: 24,
      currentTime: "",

      // tambahan
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

        // PER JAM
        this.hourlyForecast = data.hourly.time.map((t, i) => ({
          time: t.split("T")[1].slice(0, 5),
          temp: data.hourly.temperature_2m[i],
        }));

        // PER HARI (14 hari, calendar-like)
        this.dailyForecast = data.daily.time.map((t, i) => {
          const d = new Date(t);
          return {
            dayName: this.formatDayName(d, i),
            date: d.toLocaleDateString("id-ID", { day: "numeric", month: "short" }),
            high: data.daily.temperature_2m_max[i],
            low: data.daily.temperature_2m_min[i],
          };
        });

        // SUMMARY
        this.todayTemp = this.hourlyForecast[0]?.temp ?? 26;
        this.todayHigh = data.daily.temperature_2m_max[0];
        this.todayLow = data.daily.temperature_2m_min[0];

        // Extra info
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

<style scoped>
#app {
  width: 100%;
  height: 100vh;
  background: linear-gradient(150deg, #6db8ff, #0b5aa8);
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 26px 12px 40px;
  overflow-y: auto;
  font-family: "Inter", sans-serif;
}

/* Container */
.weather-card-main {
  width: 100%;
  max-width: 1200px; /* Batas maksimal untuk desktop */
}

/* Title */
.title-main {
  color: white;
  font-size: 24px;
  margin-bottom: 22px;
  text-align: center;
  font-weight: 700;
  text-shadow: 0 2px 6px rgba(0, 0, 0, 0.25);
}

/* MAIN CARD */
.main-current-card {
  background: rgba(255, 255, 255, 0.95);
  padding: 26px 20px;
  border-radius: 22px;
  margin-bottom: 30px;
  text-align: center;
  backdrop-filter: blur(6px);
  border: 1.5px solid rgba(255, 255, 255, 0.5);
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.15);
  transition: 0.25s ease;
}

.main-current-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 12px 24px rgba(0, 0, 0, 0.2);
}

.city {
  font-size: 20px;
  font-weight: 700;
  margin-bottom: 6px;
  color: #222;
}

.current-temp {
  font-size: 54px;
  font-weight: 700;
  margin-bottom: 12px;
  color: #0c74c9;
}

.row {
  display: flex;
  justify-content: center;
  gap: 16px;
  font-size: 14px;
  margin-top: 10px;
  color: #555;
}

.extra-info {
  margin-top: 14px;
  display: flex;
  justify-content: center;
  gap: 35px;
}

.info-box {
  text-align: center;
  font-size: 16px;
  font-weight: 600;
  color: #333;
}

.info-box span {
  display: block;
  font-size: 12px;
  margin-top: 2px;
  color: #666;
}

/* HEADER SECTION */
.section-title-row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 12px;
  margin-top: 10px;
  align-items: center;
}

.section-title {
  font-size: 17px;
  color: white;
  font-weight: 700;
}

/* BUTTON */
.icon-btn {
  background: rgba(255, 255, 255, 0.85);
  padding: 8px 14px;
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.6);
  transition: 0.2s;
  cursor: pointer;
  font-size: 16px;
  font-weight: bold;
  margin: 0 5px;
}

.icon-btn:hover {
  background: white;
  transform: translateY(-3px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.icon-btn:active {
  transform: translateY(0);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

/* HOURLY */
.hourly-list {
  display: flex;
  gap: 12px;
  overflow-x: auto;
  padding-bottom: 10px;
}

.hourly-list::-webkit-scrollbar {
  height: 6px;
}

.hourly-list::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.35);
  border-radius: 10px;
}

.hour-card {
  min-width: 70px;
  background: rgba(255, 255, 255, 0.92);
  border-radius: 14px;
  padding: 12px 8px;
  text-align: center;
  border: 1px solid rgba(200, 200, 200, 0.25);
  transition: 0.2s;
  cursor: pointer;
}

.hour-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 16px rgba(0,0,0,0.15);
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

/* CALENDAR GRID */
.calendar-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
  gap: 12px;
  margin-top: 10px;
}

.calendar-day {
  background: rgba(255, 255, 255, 0.92);
  padding: 16px 12px;
  border-radius: 14px;
  border: 1px solid rgba(220, 220, 220, 0.4);
  text-align: center;
  transition: 0.2s;
  cursor: pointer;
}

.calendar-day:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(0,0,0,0.15);
}

.day-name {
  font-size: 14px;
  font-weight: 600;
  color: #333;
  margin-bottom: 4px;
}

.day-date {
  font-size: 12px;
  color: #666;
  margin-bottom: 8px;
}

.day-temps {
  display: flex;
  justify-content: space-between;
  font-size: 14px;
  color: #444;
}

.day-temps span {
  font-weight: 500;
}

/* LOADING */
.loading,
.error {
  text-align: center;
  color: white;
  padding: 20px;
  font-size: 15px;
}

/* FOOTER */
.footer {
  text-align: center;
  color: white;
  font-size: 14px;
  margin-top: 30px;
  padding: 10px;
  font-weight: 500;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.3);
}

/* RESPONSIVE */
@media (max-width: 768px) {
  .current-temp {
    font-size: 48px;
  }

  .extra-info {
    gap: 25px;
  }

  .hour-card {
    min-width: 65px;
  }

  .calendar-grid {
    grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
    gap: 10px;
  }

  .calendar-day {
    padding: 12px 8px;
  }

  .day-name {
    font-size: 13px;
  }

  .day-date {
    font-size: 11px;
  }

  .day-temps {
    font-size: 13px;
  }

  .row {
    flex-direction: column;
    gap: 8px;
  }

  .footer {
    font-size: 12px;
    margin-top: 20px;
  }
}

@media (max-width: 480px) {
  .current-temp {
    font-size: 40px;
  }

  .extra-info {
    gap: 15px;
    flex-direction: column;
  }

  .hour-card {
    min-width: 55px;
    padding: 10px 6px;
  }

  .calendar-grid {
    grid-template-columns: repeat(auto-fill, minmax(80px, 1fr));
    gap: 8px;
  }

  .calendar-day {
    padding: 10px 6px;
  }

  .day-name {
    font-size: 12px;
  }

  .day-date {
    font-size: 10px;
  }

  .day-temps {
    font-size: 12px;
    flex-direction: column;
    gap: 2px;
  }

  .title-main {
    font-size: 20px;
  }

  .section-title {
    font-size: 15px;
  }

  .icon-btn {
    padding: 6px 10px;
    font-size: 14px;
    margin: 0 3px;
  }

  .footer {
    font-size: 11px;
    margin-top: 15px;
  }
}
</style>