<script setup lang="ts">
import TypeTest from './components/TypeTest.vue'
</script>

<template>
  <div class="app">
    <TypeTest />
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  background-color: #0a0a1a;
  color: #e0e0ff;
  font-family: 'Orbitron', sans-serif;
  overflow-x: hidden;
}

.app {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 2rem;
  background: radial-gradient(circle at center, #1a1a3a 0%, #0a0a1a 100%);
}
</style>
