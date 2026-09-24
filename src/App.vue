<template>
  <div class="app">
    <h1>🌤️ Weather App</h1>

    <SearchBar @search="fetchWeather" />

    <!-- Chargement -->
    <LoadingMessage v-if="loading" />

    <!-- Erreur -->
    <div v-else-if="error" class="error">
      {{ error }}
    </div>

    <!-- Succès -->
    <WeatherCard v-else-if="weather" :weather="weather" />

    <!-- État initial -->
    <div v-else class="welcome">
      <p>Recherchez une ville pour voir la météo.</p>
      <p>Essayez : Bujumbura, Tokyo, Nairobi, Shanghai, Paris</p>
    </div>
  </div>
</template>

<script>
import SearchBar from './components/SearchBar.vue';
import WeatherCard from './components/WeatherCard.vue';
import LoadingMessage from './components/LoadingMessage.vue';

export default {
  name: 'App',
  components: {
    SearchBar,
    WeatherCard,
    LoadingMessage
  },
  data() {
    return {
      loading: false,
      error: null,
      weather: null
    };
  },
  methods: {
    async fetchWeather(city) {
      this.loading = true;
      this.error = null;
      this.weather = null;

      try {
        // 1. Géocodage : ville → coordonnées
        const geoUrl = `https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(city)}&count=1&language=fr`;
        const geoRes = await fetch(geoUrl);
        const geoData = await geoRes.json();

        if (!geoData.results || geoData.results.length === 0) {
          this.error = `Ville "${city}" introuvable. Vérifiez l'orthographe.`;
          return;
        }

        const { latitude, longitude, name, country } = geoData.results[0];

        // 2. Météo : coordonnées → données
        const weatherUrl = `https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&current=temperature_2m,relative_humidity_2m,wind_speed_10m,weather_code&timezone=auto`;
        const weatherRes = await fetch(weatherUrl);
        const weatherData = await weatherRes.json();

        const current = weatherData.current;

        // 3. Construire l'objet weather
        this.weather = {
          city: country ? `${name}, ${country}` : name,
          temperature: Math.round(current.temperature_2m),
          humidity: current.relative_humidity_2m,
          windSpeed: Math.round(current.wind_speed_10m),
          condition: this.getWeatherCondition(current.weather_code),
          icon: this.getWeatherIcon(current.weather_code)
        };
      } catch (err) {
        this.error = 'Erreur réseau. Vérifiez votre connexion.';
        console.error(err);
      } finally {
        this.loading = false;
      }
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
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
body {
  font-family: system-ui, sans-serif;
  background: #f0f4f8;
  color: #333;
}
.app {
  max-width: 600px;
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
}
.welcome {
  text-align: center;
  color: #888;
  padding: 40px;
}
</style>