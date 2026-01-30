<script setup lang="ts">
import TagCanvas from './TagCanvas.vue'
</script>

<template>
  <div class="home">
    <h1 class="page-title">Brainstorming ...</h1>
    <TagCanvas />
  </div>
</template>

<style scoped>
.home {
  padding: 20px;
  min-height: calc(100vh - 70px);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  gap: 20px;
}

.page-title {
  font-size: 2.5rem;
  font-weight: 700;
  color: #001d44;
  margin: 0;
  text-align: center;
  letter-spacing: -0.02em;
}

@media (max-width: 640px) {
  .page-title {
    font-size: 1.8rem;
  }
}
</style>
