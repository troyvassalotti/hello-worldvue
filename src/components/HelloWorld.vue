<script setup>
defineProps({
  msg: String
})
</script>

<template>
  <header>
    <h1>{{ msg }}</h1>
    <p class="subtitle">hahaha, get it?</p>
    <p>Hello WorldVue is a <a href="https://v3.vuejs.org/">Vue 3</a> and <a href="https://vitejs.dev/">Vite</a> app for visualizing KMZ, KML, and GPX files in a <a href="https://leafletjs.com">Leaflet</a> map. To view your map, upload a relevant track below.</p>
  </header>
</template>

<style scoped>
header {
  text-align: center;
}

.subtitle {
  font-style: italic;
}
</style>
