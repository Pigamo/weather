<script setup lang="ts">
//List contains city objects loaded from API
const cityList = ref({})
//Individual City Object chosen from cityList
const cityObj = ref({})

//City name for input purposes
const city = ref('Toronto, Ontario, CA')
//CityName is used for UI Purposes, to display the name of the City on submit, not before.
const cityName = ref('Toronto, Ontario, CA')

//UI display of error.
const error = ref(false)

//A reactive object with weather parameters, able to be set and changed during runtime, with exceptional page speed.
const conditions = reactive({
  weather: '',
  sky: '',
  clouds: 0,
  wind: {
    speed: 0,
    deg: 0,
    gust: 0,
  },
  temp: 0,
  humidity: 0,
})

// Calls the API, but API key is exposed.
// I Would mitigate with an express server backend, but considering this isn't public, it's ok.

//This specifically converts city name to coordinates, which are necessary for openweathermap.
const getGeoFromCity = async (city: string) => {
  if (city.length > 0) {
    const res = await fetch(
      `https://api.openweathermap.org/geo/1.0/direct?q=${city},&limit=5&appid=5f5604ff8a6d663ad62ad8ac5d3ba2c1`
    )
    const data = res.json()
    return data
  }
}

// fetch the weather with the given coordinates
const getWeather = async (lat: string, lon: string) => {
  const res = await fetch(
    `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=5f5604ff8a6d663ad62ad8ac5d3ba2c1&units=metric`
  )
  const data = res.json()
  return data
}

const checkMulti = async () => {
  try {
    const data = await getGeoFromCity(city.value)
    cityList.value = data
    if (city.value == '') cityList.value = {}
  } catch (err) {
    console.log(err)
  }
}
const getMultiCity = useDebounceFn(checkMulti, 200)

// Async function that takes lat and lon data from chosen city
// to call the weather API
const callWeather = async (data: any) => {
  if (data.lon != '') {
    error.value = false
    try {
      //await promise of openweather API Weather data
      const weather = await getWeather(data.lat, data.lon)

      // Setting the reactive values: Weather, sky, wind etc.
      conditions.weather = weather.weather[0].main
      conditions.sky = weather.weather[0].description
      conditions.wind = weather.wind
      conditions.temp = weather.main.temp
      conditions.humidity = weather.main.humidity
      conditions.clouds = weather.clouds.all

      // Setting the UI name to the full city name. Updates after so it doesn't disturb UI on updates.
      cityName.value = city.value

      //Double check to empty the city chosen for next time.
      cityObj.value = {}
    } catch (err) {
      error.value = true
      //originally used to set error, but choosing city makes this mostly obsolete.
    }
  }
}
// Watches city value for changes
watch(city, () => {
  error.value = false
  //Check for empty city to wipe clean
  if (city.value == '') cityList.value = {}
  // if checks completed, generate a list of cities to choose from.
  if (city.value != '' && !city.value.includes(',')) getMultiCity()
})

// Assign city data
const assignCity = (cities: object) => {
  cityObj.value = cities
  cityList.value = {}
  city.value = `${cities.name}, ${cities.state ? cities.state + ', ' : ''}${
    cities.country
  }`
}

// Call default with Toronto to enable formatting
const setOrigin = async () => {
  try {
    const data = await getGeoFromCity('Toronto')
    cityObj.value = data[0]
    console.log(cityObj.value)
    callWeather(data[0])
  } catch (err) {
    console.log(err)
  }
}
setOrigin()

//### This was originally used to scroll down to the bottom to get to the city search UI,
//### but it was too visually complicated for something so simple.

// const scrollView = () => {
//   const element = document.querySelector('#i')
//   if (element) element.scrollIntoView({ behavior: 'smooth' })
// }
</script>

<template>
  <div
    class="flex flex-wrap items-center transition-colors transition-background md:h-screen min-h-screen-sm md:p-4 px-4 py-4 justify-around overflow-hidden"
    :style="[
      `transition: background 2s; background: linear-gradient(to bottom,  rgba(200,200,200,1), rgba(255,255,255,0.5) ${conditions.clouds}%), url(./mountains.svg) no-repeat bottom; background-size:cover`,
    ]"
  >
    <div
      class="md:w-1/2 pt-8 flex flex-col items-center"
      :class="
        cityName.split(', ')[0].length > 15
          ? 'text-3xl md:text-5xl'
          : 'sm:text-8xl text-6xl md:p-8'
      "
    >
      {{
        cityName.split(', ')[0].charAt(0).toUpperCase() +
        cityName.split(', ')[0].slice(1)
      }}

      <div class="md:w-1/2 pt-8 text-3xl md:text-5xl">
        {{ cityName.split(', ')[1] ? cityName.split(', ')[1] : '' + ', ' }}
        {{ cityName.split(', ')[2] }}
      </div>
    </div>

    <div
      v-if="conditions.weather !== ''"
      class="grid grid-cols-2 grid-rows-2 items-start text-left gap-3 pt-8"
      style="width: 400px"
    >
      <div class="flex flex-col items-start md:pr-8">
        <div class="text-5xl md:text-7xl">{{ conditions.temp.toFixed() }}C</div>
        <div class="text-3xl md:text-4xl py-4 md:w-full md:max-w-40">
          {{ conditions.sky.charAt(0).toUpperCase() + conditions.sky.slice(1) }}
        </div>
      </div>
      <div
        v-if="conditions.weather === 'Clear'"
        i-carbon-sunny
        class="text-9xl"
      />
      <div
        v-else-if="conditions.weather === 'Clouds'"
        i-carbon-cloudy
        class="text-9xl"
      />
      <div
        v-else-if="
          conditions.weather === 'Rain' || conditions.weather === 'Drizzle'
        "
        i-carbon-rain
        class="text-9xl"
      />
      <div
        v-else-if="conditions.weather === 'Snow'"
        i-carbon-snow
        class="text-9xl"
      />
      <div
        v-else-if="conditions.weather === 'Thunderstorm'"
        i-carbon-thunderstorm
        class="text-9xl"
      />
      <div v-else i-carbon-fog class="text-9xl" />

      <div class="pl-1">
        <div class="text-2xl pb-4">Wind</div>
        <div class="text-md">{{ conditions.wind.speed }} km/hr</div>
        <div class="text-md">{{ conditions.wind.deg }} degrees</div>
      </div>
      <div class="pl-1">
        <h4 class="text-2xl pb-4">Humidity</h4>
        <div class="text-md">{{ conditions.humidity }}%</div>
      </div>
    </div>

    <div
      id="i"
      class="flex flex-col items-center justify-start w-full px-10 pb-10 relative h-70 overflow-hidden"
    >
      <h2 class="text-xl md:text-3xl pb-5">Search Weather By City</h2>
      <input
        @click.prevent="city = ''"
        class="w-60 text-center text-2xl p-2"
        :class="[error ? 'bg-red-200' : '']"
        v-model="city"
        type="text"
      />
      <button
        class="w-60 p-2 bg-indigo-900 text-light-200 text-xl"
        @click.prevent="callWeather(cityObj)"
      >
        Check Weather
      </button>
      <div
        class="w-60 overflow-y-scroll"
        style="background: rgba(255, 255, 255, 0.98)"
      >
        <button
          v-for="cities in cityList"
          @click.prevent="assignCity(cities)"
          class="border-gray-400 p-1 hover:bg-light-900 cursor-pointer w-full"
        >
          {{ cities.name }}, {{ cities.state ? cities.state + ', ' : '' }}
          {{ cities.country }}
        </button>
      </div>
      <div class="absolute bottom-0" v-if="error">No city with that name</div>
    </div>
  </div>
</template>
<style scoped></style>
