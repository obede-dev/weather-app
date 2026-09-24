<template>
  <div class="weather-card">
    <h2>{{ weather.city }}</h2>
    <div class="main-info">
      <span class="temp">{{ weather.temperature }}°C</span>
      <span class="icon">{{ weather.icon }}</span>
    </div>
    <p class="condition">{{ weather.condition }}</p>
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
  </div>
</template>

<script>
export default {
  name: 'WeatherCard',
  props: {
    weather: {
      type: Object,
      required: true
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
.temp {
  font-size: 3rem;
  font-weight: bold;
}
.icon {
  font-size: 3rem;
  margin-left: 15px;
}
.details {
  display: flex;
  justify-content: center;
  gap: 30px;
  margin-top: 20px;
}
.detail-item {
  display: flex;
  flex-direction: column;
}
.label {
  font-size: 0.8rem;
  opacity: 0.8;
}
</style>