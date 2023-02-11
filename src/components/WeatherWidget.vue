<script lang="ts" setup>
import {ref, watchEffect, computed, toRefs, reactive} from 'vue'
import DraggableList from "@/components/DraggableList.vue";
import CustomInput from "@/components/UI/CustomInput.vue";
import CustomButton from "@/components/UI/CustomButton.vue";

let displayDropdownMenu = ref<boolean>(false);

function toggleDropdownMenu(): void {
  displayDropdownMenu.value = !displayDropdownMenu.value
}

// Default value for the location (Moscow, RU)
interface Location {
  lat: string;
  lon: string;
}
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

const weatherData = ref<object>()
const currentCity = ref<object>([])

watchEffect(async (): Promise<void> => {
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

interface Locations {
  id: number,
  name: string,
  location: object,
}
const locations = ref<Locations[]>([])
const newLocation = ref<any>({})
const findLocation = ref<any>();
const incorrectValue = ref<boolean>(false);

if(localStorage.getItem('locations')){
  locations.value = JSON.parse(localStorage.getItem('locations') || '{}')
}

/*
* Send request to API for find a location by city name.
* If request return empty value, display the validation error,
* else data writable to the object 'locations', and set to the localStorage.
* Temp data resets to zero
*/
async function addLocation(): Promise<void> {
  if(newLocation.value.name !== undefined && newLocation.value.name.length >= 3  && locations.value.length <= 3){
    try{
      findLocation.value = await fetch(`https://openweathermap.org/data/2.5/find?q=${newLocation.value?.name}&type=like&sort=population&cnt=30&appid=439d4b804bc8187953eb36d2a8c26a02&_=1675405012256`)
          .then(res => res.json())
    }catch (e){
      console.log(e)
    }
    if(findLocation.value?.count !== 0){
      locations.value.push({
        id: Math.random(),
        name: findLocation.value?.list[0].name,
        location: findLocation.value?.list[0].coord
      })
      localStorage.setItem('locations', JSON.stringify(locations.value))
      newLocation.value.name = undefined
      incorrectValue.value = false
    } else {
      incorrectValue.value = true
    }
  } else {
    incorrectValue.value = true
  }
}

/*
* After call function changing current location and
* on main view showing new weather information
*/
function changeLocation(value: any): void{
  location.lat = value.lat
  location.lon = value.lon
  toggleDropdownMenu()
}

/*
* Rounds the value to tenths
*/

function refactorTemperatureValue(value: number = 0){
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
              @click="addLocation">
            <p style="margin-top: 6px">
              <svg xmlns="http://www.w3.org/2000/svg" data-name="Layer 1" viewBox="0 0 64 64" width="24" height="24"><polyline fill="none" stroke="#010101" stroke-miterlimit="10" stroke-width="4" points="26.76 11.62 3.19 34.55 26.76 57.48"/><polyline fill="none" stroke="#010101" stroke-miterlimit="10" stroke-width="4" points="60.81 6.52 60.81 34.55 3.19 34.55"/></svg>
            </p>
          </CustomButton>
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
      <img :src="`http://openweathermap.org/img/wn/${weatherData?.weather[0].icon}@2x.png`" alt="weather-status">
        <h1>{{refactorTemperatureValue(weatherData?.main.temp)}}&#176;C</h1>
    </div>
    <div class="weather-card__temperature-feels-like">
      <p>Feels like {{refactorTemperatureValue(weatherData?.main.feels_like)}}&#176;C. {{weatherData?.weather[0].main}}.<br> {{weatherData?.weather[0].description }}</p>
    </div>
    <div class="weather-card__weather-info">
      <div class="weather-card__weather-info-item">
        <p style="display: flex; align-items: center">
          <svg xmlns="http://www.w3.org/2000/svg" data-name="Layer 1" viewBox="0 0 512 512" width="16" height="16"><path d="M483.10449,110.97412a7.49814,7.49814,0,0,0-7.55664-3.10742,96.55022,96.55022,0,0,1-38.48584-.04395c-22.30224-4.72216-36.59277-14.52392-60.26806-30.76367-12.48591-8.56451-27.51929-18.87115-47.46564-31.00464-.02564-.01519-.05054-.03143-.07636-.04632-4.09265-2.48895-8.37567-5.04871-12.90381-7.69269-47.85449-27.94434-104.312-27.95264-167.80126-.02734a7.48978,7.48978,0,0,0-2.43982,1.72821,7.48917,7.48917,0,0,0-1.22083,1.3255L84.99951,125.21021V115.5a7.50006,7.50006,0,0,0-7.5-7.5h-6V64.07275a36,36,0,1,0-45,0V108h-6a7.50006,7.50006,0,0,0-7.5,7.5v62a7.50006,7.50006,0,0,0,7.5,7.5h6V504.5a7.50006,7.50006,0,0,0,7.5,7.5h30a7.50006,7.50006,0,0,0,7.5-7.5V185h6a7.50006,7.50006,0,0,0,7.5-7.5V167.498l60.98487,64.63922a7.487,7.487,0,0,0,10.45294.43567c41.76568-25.92279,83.44751-36.04444,123.90985-30.07386,16.57178,2.44629,27.875,6.80762,39.84228,11.42432,10.334,3.9873,21.01953,8.10986,35.625,11.31885,25.46045,5.59326,52.73047,5.76416,81.05371.51074a283.47213,283.47213,0,0,0,43.85694-12.0332,7.50014,7.50014,0,0,0,3.75-2.96582,89.47246,89.47246,0,0,0-1.37061-99.77979ZM56.49951,497h-15V185h15Zm13.5-327h-42V123h42Zm-28.5-62V71.20862a35.93487,35.93487,0,0,0,15,0V108Zm7.5-51a21,21,0,1,1,21-21A21.02359,21.02359,0,0,1,48.99951,57Zm38.18756,90.95874,35.75-50.0661a182.42064,182.42064,0,0,0,4.26874,92.48291Zm64.02759,62.29346a167.97911,167.97911,0,0,1,.06305-148.62848A183.63729,183.63729,0,0,1,167,138.39648,183.91108,183.91108,0,0,1,151.21466,210.2522Zm89.79315-89.10474a181.11788,181.11788,0,0,1-16.71167,67.88489,211.454,211.454,0,0,0-56.19995,19.72595A198.54939,198.54939,0,0,0,181.999,138.60352,198.99365,198.99365,0,0,0,161.918,48.91162q31.94477-13.018,61.0003-15.75055A183.80973,183.80973,0,0,1,241.00781,121.14746ZM331.00244,131.293a202.94878,202.94878,0,0,1-12.86816,65.79724,167.411,167.411,0,0,0-35.59668-9.43005,170.62892,170.62892,0,0,0-24.96192-1.83448q-7.913,0-15.85778.74213a195.5744,195.5744,0,0,0,14.27331-64.71527,198.43278,198.43278,0,0,0-16.8916-89.41742,137.525,137.525,0,0,1,69.68506,18.83343c3.66742,2.14142,7.16894,4.22632,10.54608,6.26618A202.88068,202.88068,0,0,1,331.00244,131.293Zm68.60419,83.24432a175.59461,175.59461,0,0,1-40.57343-3.94549,189.74711,189.74711,0,0,1-26.88415-8.15692A218.60025,218.60025,0,0,0,338.79291,69.67c11.4953,7.39875,21.09192,13.98029,29.51617,19.75873,11.45612,7.85809,21.08234,14.44464,30.69263,19.79016a205.479,205.479,0,0,1,.60492,105.31842Zm73.61456-14.07245a266.3507,266.3507,0,0,1-39.08935,10.54q-9.5312,1.76806-18.86341,2.6518a220.0671,220.0671,0,0,0,1.11713-96.26874,119.2716,119.2716,0,0,0,17.569,5.10962,111.06434,111.06434,0,0,0,39.34521.95312,74.40245,74.40245,0,0,1-.07861,77.01416Z"/></svg>
          {{refactorTemperatureValue(weatherData?.wind.speed)}}m/s SSE
        </p>
      </div>
      <div class="weather-card__weather-info-item">
        <p style="display: flex; align-items: center">
          <svg xmlns="http://www.w3.org/2000/svg" data-name="Layer 1" viewBox="0 0 24 24" width="18" height="18"><path d="M12 5a10 10 0 0 0-8.66 15 1 1 0 0 0 1.74-1A7.92 7.92 0 0 1 4 15a8 8 0 1 1 14.93 4 1 1 0 0 0 .37 1.37 1 1 0 0 0 1.36-.37A10 10 0 0 0 12 5Zm2.84 5.76-1.55 1.54A2.91 2.91 0 0 0 12 12a3 3 0 1 0 3 3 2.9 2.9 0 0 0-.3-1.28l1.55-1.54a1 1 0 0 0 0-1.42 1 1 0 0 0-1.41 0ZM12 16a1 1 0 0 1 0-2 1 1 0 0 1 .7.28 1 1 0 0 1 .3.72 1 1 0 0 1-1 1Z"/></svg>
          {{refactorTemperatureValue(weatherData?.main.pressure)}}hPa
        </p>
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
.weather-card__header-setting path, .weather-card__header-setting svg{
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
    transform-origin: 0 13px;
  }
}
</style>
