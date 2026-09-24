<template>
  <div class="app">
    <h1>🌤️ Weather App</h1>

    <!-- Historique de recherche -->
    <div v-if="history.length > 0" class="chips-row">
      <span class="chips-label">🕘 Récents :</span>
      <button
        v-for="city in history"
        :key="city"
        @click="fetchWeather(city)"
        class="chip"
      >
        {{ city }}
      </button>
    </div>

    <!-- Favoris -->
    <div v-if="favorites.length > 0" class="chips-row">
      <span class="chips-label">⭐ Favoris :</span>
      <button
        v-for="city in favorites"
        :key="city"
        @click="fetchWeather(city)"
        class="chip chip-fav"
      >
        {{ city }}
      </button>
    </div>

    <!-- Barre de recherche -->
    <SearchBar @search="fetchWeather" />

    <!-- État : chargement -->
    <LoadingMessage v-if="loading" />

    <!-- État : erreur -->
    <div v-else-if="error" class="error">
      ⚠️ {{ error }}
    </div>

    <!-- État : succès -->
    <div v-else-if="weather">
      <WeatherCard
        :weather="weather"
        :unit="unit"
        :is-favorite="isFavorite()"
        @toggle-unit="unit = unit === 'C' ? 'F' : 'C'"
        @toggle-favorite="toggleFavorite"
      />
    </div>

    <!-- État initial -->
    <div v-else class="welcome">
      <p>Recherchez une ville pour voir la météo.</p>
      <p class="hint">Essayez : Bujumbura, Tokyo, Nairobi, Shanghai, Paris</p>
    </div>
  </div>
</template>

<script>
import SearchBar from './components/SearchBar.vue';
import WeatherCard from './components/WeatherCard.vue';
import LoadingMessage from './components/LoadingMessage.vue';

export default {
  name: 'App',
  components: { SearchBar, WeatherCard, LoadingMessage },
  data() {
    return {
      loading: false,
      error: null,
      weather: null,
      unit: 'C',
      history: [],
      favorites: []
    };
  },
  methods: {
    async fetchWeather(city) {
      this.loading = true;
      this.error = null;
      this.weather = null;

      try {
        // 1. Géocodage
        const geoUrl = `https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(city)}&count=1&language=fr`;
        const geoRes = await fetch(geoUrl);
        const geoData = await geoRes.json();

        if (!geoData.results || geoData.results.length === 0) {
          this.error = `Ville "${city}" introuvable. Vérifiez l'orthographe.`;
          this.loading = false;
          return;
        }

        const { latitude, longitude, name, country } = geoData.results[0];

        // 2. Météo + prévisions
        const weatherUrl = `https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&current=temperature_2m,relative_humidity_2m,wind_speed_10m,weather_code&daily=weather_code,temperature_2m_max,temperature_2m_min&timezone=auto&forecast_days=5`;
        const weatherRes = await fetch(weatherUrl);
        const weatherData = await weatherRes.json();

        const current = weatherData.current;
        const daily = weatherData.daily;

        // 3. Prévisions 5 jours
        const forecast = daily.time.map((date, i) => ({
          date: new Date(date).toLocaleDateString('fr-FR', {
            weekday: 'short',
            day: 'numeric'
          }),
          max: Math.round(daily.temperature_2m_max[i]),
          min: Math.round(daily.temperature_2m_min[i]),
          icon: this.getWeatherIcon(daily.weather_code[i])
        }));

        // 4. Objet météo
        const displayName = country ? `${name}, ${country}` : name;
        this.weather = {
          city: displayName,
          temperature: Math.round(current.temperature_2m),
          humidity: current.relative_humidity_2m,
          windSpeed: Math.round(current.wind_speed_10m),
          condition: this.getWeatherCondition(current.weather_code),
          icon: this.getWeatherIcon(current.weather_code),
          forecast
        };

        // 5. Historique (sans doublons, max 5)
        this.history = [
          displayName,
          ...this.history.filter(c => c !== displayName)
        ].slice(0, 5);
      } catch (err) {
        this.error = 'Erreur réseau. Vérifiez votre connexion.';
        console.error(err);
      } finally {
        this.loading = false;
      }
    },

    toggleFavorite() {
      if (!this.weather) return;
      const city = this.weather.city;
      if (this.favorites.includes(city)) {
        this.favorites = this.favorites.filter(c => c !== city);
      } else {
        this.favorites.push(city);
      }
    },

    isFavorite() {
      return this.weather && this.favorites.includes(this.weather.city);
    },

    getWeatherCondition(code) {
      const conditions = {
        0: 'Ciel dégagé', 1: 'Principalement dégagé', 2: 'Partiellement nuageux', 3: 'Couvert',
        45: 'Brouillard', 48: 'Brouillard givrant',
        51: 'Bruine légère', 53: 'Bruine modérée', 55: 'Bruine dense',
        61: 'Pluie légère', 63: 'Pluie modérée', 65: 'Pluie forte',
        71: 'Neige légère', 73: 'Neige modérée', 75: 'Neige forte',
        80: 'Averses légères', 81: 'Averses modérées', 82: 'Averses violentes',
        95: 'Orage', 96: 'Orage avec grêle'
      };
      return conditions[code] || 'Conditions inconnues';
    },

    getWeatherIcon(code) {
      if (code === 0) return '☀️';
      if (code <= 3) return '⛅';
      if (code <= 48) return '🌫️';
      if (code <= 67) return '🌧️';
      if (code <= 77) return '❄️';
      if (code <= 82) return '🌦️';
      if (code >= 95) return '⛈️';
      return '🌤️';
    }
  }
};
</script>

<style>
* { margin: 0; padding: 0; box-sizing: border-box; }
body {
  font-family: system-ui, -apple-system, sans-serif;
  background: linear-gradient(135deg, #e0eafc, #cfdef3);
  min-height: 100vh;
  color: #333;
}
.app {
  max-width: 650px;
  margin: 40px auto;
  padding: 30px;
  background: white;
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.1);
}
h1 {
  text-align: center;
  margin-bottom: 25px;
  color: #4a90d9;
}
.error {
  background: #ffe5e5;
  color: #c0392b;
  padding: 15px;
  border-radius: 8px;
  text-align: center;
  border-left: 4px solid #c0392b;
}
.welcome { text-align: center; color: #888; padding: 40px; }
.welcome .hint { font-size: 0.9rem; margin-top: 10px; color: #aaa; }

.chips-row {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  align-items: center;
  margin-bottom: 15px;
}
.chips-label { font-size: 0.85rem; color: #888; }
.chip {
  padding: 5px 12px;
  background: #eef0ff;
  color: #4a90d9;
  border: none;
  border-radius: 15px;
  cursor: pointer;
  font-size: 0.85rem;
  transition: background 0.2s;
}
.chip:hover { background: #dbe4ff; }
.chip-fav { background: #fff8e1; color: #b8860b; }
.chip-fav:hover { background: #fff3c4; }
</style>