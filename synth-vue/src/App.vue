<script setup>
import Synth from './components/Synth.vue'
</script>

<template>
  <div class="app">
    <h1>JS Synth</h1>
    <Synth />
  </div>
</template>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: 'Segoe UI', system-ui, sans-serif;
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  color: #e0e0e0;
}

h1 {
  margin-bottom: 20px;
  font-weight: 300;
  letter-spacing: 4px;
  text-transform: uppercase;
  color: #00d9ff;
  text-shadow: 0 0 20px rgba(0, 217, 255, 0.5);
}

#app {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}
</style>
