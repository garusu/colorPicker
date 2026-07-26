<script setup>
  import { Color } from './components/color.js';
  import Picker from './components/Picker.vue';

  const color = new Color();
</script>

<template>
  <div id="container" :style="{backgroundColor: `rgb(${color.r} ${color.g} ${color.b})`}">
    <h1>{{ color.hex }}</h1>
    <Picker :color="color"></Picker>
  </div>
</template>

<style scoped>
  h1 {
    margin: 10px 0;
  }
  #container {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    height: 100vh;
    touch-action: none;
  }
</style>
