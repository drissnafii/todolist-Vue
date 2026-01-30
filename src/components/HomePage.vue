<script setup lang="ts">
import TagCanvas from './TagCanvas.vue'
</script>

<template>
  <div class="home">
    <TagCanvas />
  </div>
</template>

<style scoped>
.home {
  padding: 20px;
  min-height: calc(100vh - 70px);
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
