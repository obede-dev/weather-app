<template>
  <div class="loading">
    <div class="spinner"></div>
    <p>{{ message }}</p>
  </div>
</template>

<script>
export default {
  name: 'LoadingMessage',
  props: {
    message: {
      type: String,
      default: 'Chargement des données météo...'
    }
  }
};
</script>

<style scoped>
.loading {
  text-align: center;
  padding: 40px;
}
.spinner {
  width: 50px;
  height: 50px;
  border: 5px solid #eee;
  border-top-color: #4a90d9;
  border-radius: 50%;
  margin: 0 auto 15px;
  animation: spin 1s linear infinite;
}
@keyframes spin {
  to { transform: rotate(360deg); }
}
</style>