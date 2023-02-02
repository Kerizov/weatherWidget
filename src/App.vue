<script lang="ts">

import { defineComponent } from 'vue';
import WeatherWidget from '@/components/WeatherWidget.vue';

export default defineComponent({
  name: 'App',
  components: {
    WeatherWidget
  }
});
</script>
<template>
  <weather-widget
      APIKey="d136d9defb07d07b334ea13f38c861d4"
  />
</template>

<style lang="scss">
*{
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
