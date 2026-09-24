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
    return { cityInput: '' };
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
.search-bar { display: flex; gap: 10px; margin-bottom: 20px; }
input {
  flex: 1;
  padding: 12px 16px;
  border: 2px solid #e0e0e0;
  border-radius: 10px;
  font-size: 1rem;
  transition: border-color 0.2s;
}
input:focus { outline: none; border-color: #4a90d9; }
button {
  padding: 12px 24px;
  background: #4a90d9;
  color: white;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  font-size: 1rem;
  font-weight: 600;
  transition: background 0.2s;
}
button:hover:not(:disabled) { background: #3a7bc0; }
button:disabled { background: #ccc; cursor: not-allowed; }
</style>