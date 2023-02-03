<script lang="ts" setup>
import {ref, watchEffect, computed, toRefs, reactive} from 'vue'
import DraggableList from "@/components/DraggableList.vue";
import CustomInput from "@/components/UI/CustomInput.vue";
import CustomButton from "@/components/UI/CustomButton.vue";

let displayDropdownMenu = ref<boolean>(false);

function toggleDropdownMenu() {
   displayDropdownMenu.value = !displayDropdownMenu.value
}

interface Location {
  lat: string;
  lon: string;
}
// Default value for the location
const location = reactive<Location>({
  lat: '55.8168',
  lon: '37.7982',
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

const weatherData = ref<any>()
const currentCity = ref<any>([])


watchEffect(async () => {
  const currentLocation = toRefs(location)
  try{
    weatherData.value = await fetch(`https://api.openweathermap.org/data/2.5/weather?lat=${currentLocation.lat.value}&lon=${currentLocation.lon.value}&appid=d136d9defb07d07b334ea13f38c861d4&units=metric`)
        .then(res => res.json())
    currentCity.value = await fetch(`https://api.openweathermap.org/geo/1.0/reverse?lat=${currentLocation.lat.value}&lon=${currentLocation.lon.value}&limit=1&appid=d136d9defb07d07b334ea13f38c861d4`)
        .then(res => res.json())
  } catch (e){
    console.log(e)
  }
})

const locations = ref<any>([])
const newLocation = ref<any>({})
const findLocation = ref<any>();
const incorrectValue = ref<boolean>(false);

if(localStorage.getItem('locations')){
  locations.value = JSON.parse(localStorage.getItem('locations') || '{}')
}

async function addLocation(){
  if(newLocation.value.name !== undefined && locations.value.length <= 3){
    try{
      findLocation.value = await fetch(`https://openweathermap.org/data/2.5/find?q=${newLocation.value.name}&type=like&sort=population&cnt=30&appid=439d4b804bc8187953eb36d2a8c26a02&_=1675405012256`)
          .then(res => res.json())
    }catch (e){
      console.log(e)
    }
    if(findLocation.value?.count === 0) incorrectValue.value = true
    locations.value.push({
      id: Math.random(),
      name: findLocation.value?.list[0].name,
      location: findLocation.value?.list[0].coord
    })
    localStorage.setItem('locations', JSON.stringify(locations.value))
    newLocation.value.name = undefined
  }
}

function changeLocation(value: any){
  location.lat = value.lat
  location.lon = value.lon
  toggleDropdownMenu()
}

function refactorTemperatureValue(value: number = 0) {
  return computed(() => +value?.toFixed(1));
}
</script>

<template>
  <div class="weather-card">
    <div class="dropdown-menu"
         :style="!displayDropdownMenu ? {'display': 'none'} : ''">
        <DraggableList
            :list="locations"
            @updateLocation="changeLocation"/>
        <div style="display: flex">
          <div style="display: block">
            <label style="font-weight: bolder">Add Location</label>
            <CustomInput
              @keyup.enter="addLocation"
              v-model.trim="newLocation.name"
              :class="incorrectValue ? 'danger' : ''"
              placeholder="City"
              label="Add Location"/>
          </div>
          <CustomButton
              @click="addLocation"><img
              style="width: 24px"
              src="@/assets/enter-key.svg"
              alt="enter-key"></CustomButton>
        </div>
    </div>
    <div class="weather-card__header">
      <div class="weather-card__header-title">
        <template v-if="displayDropdownMenu">
            Settings
        </template>
        <template v-else>
            {{ currentCity[0]?.name }}, {{weatherData?.sys.country}}
        </template>
      </div>
      <button
          @click="toggleDropdownMenu"
          :class="displayDropdownMenu ? 'active' : ''"
          class="weather-card__header-setting">
          <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="17px" height="16px">
            <path fill-rule="evenodd"  fill="rgb(0, 0, 0)" d="M-0.000,-0.000 L17.000,-0.000 L17.000,2.000 L-0.000,2.000 L-0.000,-0.000 Z"/>
            <path fill-rule="evenodd"  fill="rgb(0, 0, 0)" d="M-0.000,7.000 L17.000,7.000 L17.000,9.000 L-0.000,9.000 L-0.000,7.000 Z"/>
            <path fill-rule="evenodd"  fill="rgb(0, 0, 0)" d="M-0.000,14.000 L17.000,14.000 L17.000,16.000 L-0.000,16.000 L-0.000,14.000 Z"/>
          </svg>
      </button>
    </div>
    <div class="weather-card__temperature">
      <template v-if="weatherData?.weather[0].icon === '01d'">
        <img src="@/assets/weather-status/01d.png" alt="sun">
      </template>
      <template v-if="weatherData?.weather[0].icon === '01n'">
        <img src="@/assets/weather-status/01n.png" alt="sun">
      </template>
      <template v-if="weatherData?.weather[0].icon === '02d'">
        <img src="@/assets/weather-status/02d.png" alt="sun">
      </template>
      <template v-if="weatherData?.weather[0].icon === '02n'">
        <img src="@/assets/weather-status/02n.png" alt="sun">
      </template>
      <template v-if="weatherData?.weather[0].icon === '03d'">
        <img src="@/assets/weather-status/03d.png" alt="sun">
      </template>
      <template v-if="weatherData?.weather[0].icon === '03n'">
        <img src="@/assets/weather-status/03n.png" alt="sun">
      </template>
      <template v-if="weatherData?.weather[0].icon === '04d'">
        <img src="@/assets/weather-status/04d.png" alt="sun">
      </template>
      <template v-if="weatherData?.weather[0].icon === '04n'">
        <img src="@/assets/weather-status/04n.png" alt="sun">
      </template>
      <template v-if="weatherData?.weather[0].icon === '09d'">
        <img src="@/assets/weather-status/09d.png" alt="sun">
      </template>
      <template v-if="weatherData?.weather[0].icon === '09n'">
        <img src="@/assets/weather-status/09n.png" alt="sun">
      </template>
      <template v-if="weatherData?.weather[0].icon === '10d'">
        <img src="@/assets/weather-status/10d.png" alt="sun">
      </template>
      <template v-if="weatherData?.weather[0].icon === '10n'">
        <img src="@/assets/weather-status/10n.png" alt="sun">
      </template>
      <template v-if="weatherData?.weather[0].icon === '11d'">
        <img src="@/assets/weather-status/11d.png" alt="sun">
      </template>
      <template v-if="weatherData?.weather[0].icon === '11n'">
        <img src="@/assets/weather-status/11n.png" alt="sun">
      </template>
      <template v-if="weatherData?.weather[0].icon === '13d'">
        <img src="@/assets/weather-status/13d.png" alt="sun">
      </template>
      <template v-if="weatherData?.weather[0].icon === '13n'">
        <img src="@/assets/weather-status/13n.png" alt="sun">
      </template>
      <template v-if="weatherData?.weather[0].icon === '50d'">
        <img src="@/assets/weather-status/50d.png" alt="sun">
      </template>
      <template v-if="weatherData?.weather[0].icon === '50n'">
        <img src="@/assets/weather-status/50n.png" alt="sun">
      </template>
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
  position: relative;
  z-index: 0;
  &__header{
    display: flex;
    justify-content: space-between;
  }
  &__header-title{
    font-size: 15px;
    font-weight: bolder;
    z-index: 10;
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
.dropdown-menu{
  display: flex;
  flex-direction: column;
  width: 260px;
  height: 350px;
  padding: 50px 20px;
  box-shadow: 1px 1px 7px 1px rgba(83, 83, 83, 0.5);
  background-color: #fff;
  position: absolute;
  top: 0;
  left: 0;
}
path, svg{
  position: sticky;
  z-index: 10;
  transition: transform 0.25s;
}
.danger{
  font-size: 16px;
  color: rgba(248, 0, 0, 0.56);
}
.active{
  & path:nth-of-type(1){
    transform: rotate(45deg);
    transform-origin: -1px 3px;
  }
  & path:nth-of-type(2){
    display: none;
  }
  & path:nth-of-type(3){
    transform: rotate(-45deg);
    transform-origin: 0px 13px;
  }
}
</style>
