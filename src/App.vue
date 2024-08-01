<script setup lang="ts">
import HelloWorld from './components/HelloWorld.vue'
import TEditor from './components/TEditor.vue'
</script>

<template>
  <div style="height: calc(100vh - 4rem);">
    <TEditor />
  </div>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}

.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}

.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
