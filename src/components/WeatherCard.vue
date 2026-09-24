<template>
  <div class="weather-card">
    <h2>{{ weather.city }}</h2>

    <div class="main-info">
      <span class="temp">{{ displayTemp }}{{ unitSymbol }}</span>
      <span class="icon">{{ weather.icon }}</span>
    </div>

    <p class="condition">{{ weather.condition }}</p>

    <button class="unit-toggle" @click="$emit('toggle-unit')">
      Afficher en {{ unit === 'C' ? '°F' : '°C' }}
    </button>

    <div class="details">
      <div class="detail-item">
        <span class="label">Humidité</span>
        <span class="value">{{ weather.humidity }}%</span>
      </div>
      <div class="detail-item">
        <span class="label">Vent</span>
        <span class="value">{{ weather.windSpeed }} km/h</span>
      </div>
    </div>

    <!-- Prévisions 5 jours -->
    <div v-if="weather.forecast" class="forecast">
      <h3>Prévisions 5 jours</h3>
      <div class="forecast-grid">
        <div
          v-for="day in weather.forecast"
          :key="day.date"
          class="forecast-day"
        >
          <div class="fc-date">{{ day.date }}</div>
          <div class="fc-icon">{{ day.icon }}</div>
          <div class="fc-temps">
            <span class="fc-max">{{ convert(day.max) }}°</span>
            <span class="fc-min">{{ convert(day.min) }}°</span>
          </div>
        </div>
      </div>
    </div>
  </div>

  <button class="fav-btn" @click="$emit('toggle-favorite')">
    {{ isFavorite ? '⭐ Retirer des favoris' : '☆ Ajouter aux favoris' }}
  </button>
</template>

<script>
export default {
  name: 'WeatherCard',
  props: {
    weather: { type: Object, required: true },
    unit: { type: String, default: 'C' },
    isFavorite: { type: Boolean, default: false }
  },
  emits: ['toggle-unit', 'toggle-favorite'],
  computed: {
    displayTemp() {
      return this.convert(this.weather.temperature);
    },
    unitSymbol() {
      return this.unit === 'F' ? '°F' : '°C';
    }
  },
  methods: {
    convert(celsius) {
      if (this.unit === 'F') {
        return Math.round((celsius * 9) / 5 + 32);
      }
      return celsius;
    }
  }
};
</script>

<style scoped>
.weather-card {
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  padding: 30px;
  border-radius: 16px;
  text-align: center;
}
.weather-card h2 { font-size: 1.5rem; margin-bottom: 10px; }
.main-info { display: flex; justify-content: center; align-items: center; gap: 15px; }
.temp { font-size: 3rem; font-weight: bold; }
.icon { font-size: 3rem; }
.condition { margin-top: 5px; opacity: 0.9; }

.unit-toggle {
  margin-top: 15px;
  padding: 8px 16px;
  background: rgba(255,255,255,0.2);
  color: white;
  border: 1px solid rgba(255,255,255,0.4);
  border-radius: 20px;
  cursor: pointer;
  font-size: 0.85rem;
  transition: background 0.2s;
}
.unit-toggle:hover { background: rgba(255,255,255,0.3); }

.details {
  display: flex;
  justify-content: center;
  gap: 40px;
  margin-top: 25px;
  padding-top: 20px;
  border-top: 1px solid rgba(255,255,255,0.3);
}
.detail-item { display: flex; flex-direction: column; }
.label { font-size: 0.8rem; opacity: 0.8; margin-bottom: 3px; }
.value { font-size: 1.1rem; font-weight: 600; }

.forecast {
  margin-top: 25px;
  padding-top: 20px;
  border-top: 1px solid rgba(255,255,255,0.3);
}
.forecast h3 { font-size: 1rem; margin-bottom: 15px; }
.forecast-grid {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 8px;
}
.forecast-day {
  background: rgba(255,255,255,0.15);
  padding: 10px 5px;
  border-radius: 10px;
  text-align: center;
}
.fc-date { font-size: 0.72rem; text-transform: capitalize; opacity: 0.9; }
.fc-icon { font-size: 1.4rem; margin: 5px 0; }
.fc-temps { font-size: 0.8rem; }
.fc-max { font-weight: bold; }
.fc-min { opacity: 0.7; margin-left: 4px; }

.fav-btn {
  width: 100%;
  margin-top: 12px;
  padding: 12px;
  background: #fff8e1;
  color: #b8860b;
  border: 2px solid #ffe082;
  border-radius: 10px;
  cursor: pointer;
  font-size: 0.95rem;
  font-weight: 600;
  transition: background 0.2s;
}
.fav-btn:hover { background: #fff3c4; }

@media (max-width: 500px) {
  .forecast-grid { grid-template-columns: repeat(3, 1fr); }
  .temp { font-size: 2.2rem; }
  .icon { font-size: 2.2rem; }
}
</style>