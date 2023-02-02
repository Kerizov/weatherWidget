<script lang="ts" setup>
import {ref, watchEffect, computed, toRefs, reactive} from 'vue'

interface Location {
  lat: string;
  lon: string;
}
// Default value for the location
const location = reactive<Location>({
  lat: '0',
  lon: '0',
})

// Options and request to access current location
const options: {enableHighAccuracy: boolean, timeout: number, maximumAge: number} = {
  enableHighAccuracy: true,
  timeout: 5000,
  maximumAge: 0
};

function success(pos: any): void {
  const crd = pos.coords;

  location.lat = crd.latitude
  location.lon = crd.longitude
}

function error(err: any): void {
  console.warn(`ERROR(${err.code}): ${err.message}`);
}
// Request the current user’s location
(async function getLocation(): Promise<void> {
  await navigator.geolocation.getCurrentPosition(success, error, options)
})()

const props = defineProps({
  "APIKey": String,
})

const weatherData = ref<any>()
const currentCity = ref<any>([])

watchEffect(async () => {
  const currentLocation = toRefs(location)
  try{
    weatherData.value = await fetch(`https://api.openweathermap.org/data/2.5/weather?lat=${currentLocation.lat.value}&lon=${currentLocation.lon.value}&appid=${props.APIKey}&units=metric`)
        .then(res => res.json())
    currentCity.value = await fetch(`https://api.openweathermap.org/geo/1.0/reverse?lat=${currentLocation.lat.value}&lon=${currentLocation.lon.value}&limit=1&appid=${props.APIKey}`)
        .then(res => res.json())
  } catch (e){
    console.log(e)
  }
})

function refactorTemperatureValue(value: number = 0) {
  return computed(() => +value?.toFixed(1));
}
</script>

<template>
  <div class="weather-card">
    <div class="weather-card__header">
      <div class="weather-card__header-location">
        {{ currentCity[0]?.name }}, {{weatherData?.sys.country}}
      </div>
      <button class="weather-card__header-setting">
        <img src="@/assets/setting.svg" alt="icon-setting">
      </button>
    </div>
    <div class="weather-card__temperature">
        <img src="@/assets/weather-status/sun.png" alt="sun">
        <h1>{{refactorTemperatureValue(weatherData?.main.temp)}}&#176;C</h1>
    </div>
    <div class="weather-card__temperature-feels-like">
      <p>Feels like {{refactorTemperatureValue(weatherData?.main.feels_like)}}&#176;C. Broken clouds. Light breeze</p>
    </div>
    <div class="weather-card__weather-info">
      <div class="weather-card__weather-info-item">
        <p><img src="@/assets/windsock.svg" alt="windstock">{{refactorTemperatureValue(weatherData?.wind.speed)}}m/s SSE</p>
      </div>
      <div class="weather-card__weather-info-item">
        <p><img src="@/assets/dashboard.svg" alt="pressure">{{refactorTemperatureValue(weatherData?.main.pressure)}}hPa</p>
      </div>
      <div class="weather-card__weather-info-item">
        <p>Humidity: {{refactorTemperatureValue(weatherData?.main.humidity)}}%</p>
      </div>
      <div class="weather-card__weather-info-item">
        <p>Dew point: {{refactorTemperatureValue(weatherData?.main.temp - ((100 - weatherData?.main.humidity)/5))}}&#176;C</p>
      </div>
      <div class="weather-card__weather-info-item">
        <p>Visibility: {{weatherData?.visibility / 1000}}km</p>
      </div>
    </div>
  </div>
</template>

<style scoped lang="scss">
.weather-card{
  display: flex;
  flex-direction: column;
  width: 260px;
  height: 350px;
  margin: 15px;
  padding: 10px;
  box-shadow: 1px 1px 7px 1px rgba(83, 83, 83, 0.5);
  &__header{
    display: flex;
    justify-content: space-between;
  }
  &__header-location{
    font-size: 15px;
    font-weight: bolder;
  }
  &__header-setting{
    border: none;
    outline: none;
    width: 24px;
    height: 24px;
    cursor: pointer;
  }
  &__temperature{
    display: flex;
    text-align: center;
    flex-direction: column;
    margin: 0 auto;
    width: 100px;
    height: auto;
    &-feels-like{
      margin-top: 10px;
    }
  }
  &__weather-info{
    display: grid;
    grid-template-columns: 1fr 1fr;
    grid-gap: 2px;
    font-size: 15px;
    margin-top: 10px;
    &-item{
      margin-top: 10px;
      white-space: nowrap;
      & img{
        width: 16px;
        height: 16px;
      }
    }
  }
}
</style>
