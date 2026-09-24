<template>
  <div class="search-bar">
    <input
      v-model="cityInput"
      type="text"
      placeholder="Entrez une ville (ex: Bujumbura, Tokyo...)"
      @keyup.enter="onSearch"
    />
    <button @click="onSearch" :disabled="!cityInput.trim()">
      Rechercher
    </button>
  </div>
</template>

<script>
export default {
  name: 'SearchBar',
  data() {
    return {
      cityInput: ''
    };
  },
  methods: {
    onSearch() {
      if (this.cityInput.trim() === '') return;
      this.$emit('search', this.cityInput.trim());
      this.cityInput = '';
    }
  }
};
</script>

<style scoped>
.search-bar {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}
input {
  flex: 1;
  padding: 12px;
  border: 2px solid #ddd;
  border-radius: 8px;
  font-size: 1rem;
}
button {
  padding: 12px 24px;
  background: #4a90d9;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 1rem;
}
button:disabled {
  background: #ccc;
  cursor: not-allowed;
}
</style>