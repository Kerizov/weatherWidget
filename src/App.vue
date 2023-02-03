<script lang="ts">

import { defineComponent } from 'vue';
import WeatherWidget from '@/components/WeatherWidget.vue';
import DraggableList from '@/components/DraggableList.vue';

export default defineComponent({
  name: 'App',
  components: {
    WeatherWidget,
    DraggableList
  }
});
</script>

<template>
  <weather-widget/>
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
