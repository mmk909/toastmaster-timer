<template>
  <div class="app">
    <TimerPage />
  </div>
</template>

<script setup>
import TimerPage from './components/TimerPage.vue'
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body, html {
  height: 100%;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, sans-serif;
}

#app {
  height: 100%;
}
</style>
