<script setup>
defineProps({
  msg: String
})
</script>

<template>
  <footer>
    <h2>{{ msg }}</h2>
    <p>This app doesn't use a database and I have no way of knowing details about the files you upload. I've restricted you to only uploading specific file types and that's that. All files are accessed in the browser and loaded as Object URLs in the map. When you close this window, those Object URLs are revoked (as is the nature of Object URLs) and you can safely reload this app at square one.</p>
  </footer>
</template>

<style scoped>

</style>