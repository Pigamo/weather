<template>
  <div
    class="flex flex-wrap items-center transition-colors transition-background min-h-screen md:h-screen md:min-h-screen-sm md:p-8 px-8 py-4 justify-around"
    :style="[
      `transition: background 2s; background: linear-gradient(to bottom,  rgba(200,200,200,1), rgba(255,255,255,0.5) ${conditions.clouds}%), url(./mountains.svg) no-repeat bottom; background-size:cover`,
    ]"
  >
    <div class="md:text-8xl text-5xl md:p-8 pt-8">
      {{ cityName.charAt(0).toUpperCase() + cityName.slice(1) }}
    </div>

    <div
      v-if="conditions.weather != ''"
      class="flex flex-col items-center justify-around w-full md:w-auto"
    >
      <div
        class="flex items-center md:items-start w-full justify-evenly md:justify-between text-left"
      >
        <div class="flex flex-col items-start md:pr-8">
          <div class="text-5xl md:text-7xl">
            {{ conditions.temp.toFixed() }}C
          </div>
          <div class="text-3xl md:text-4xl py-4 md:w-full md:max-w-40">
            {{
              conditions.sky.charAt(0).toUpperCase() + conditions.sky.slice(1)
            }}
          </div>
        </div>
        <div
          v-if="conditions.weather == 'Clear'"
          i-carbon-sunny
          class="text-9xl"
        ></div>
        <div
          v-else-if="conditions.weather == 'Clouds'"
          i-carbon-cloudy
          class="text-9xl"
        ></div>
        <div
          v-else-if="
            conditions.weather == 'Rain' || conditions.weather == 'Drizzle'
          "
          i-carbon-rain
          class="text-9xl"
        ></div>
        <div
          v-else-if="conditions.weather == 'Snow'"
          i-carbon-snow
          class="text-9xl"
        ></div>
        <div
          v-else-if="conditions.weather == 'Thunderstorm'"
          i-carbon-thunderstorm
          class="text-9xl"
        ></div>
        <div v-else i-carbon-fog class="text-9xl"></div>
      </div>
      <div class="flex items-start flex-wrap w-full justify-evenly pt-4">
        <div>
          <div class="text-2xl pb-4">Wind</div>
          <div class="text-md">{{ conditions.wind.speed }} km/hr</div>
          <div class="text-md">{{ conditions.wind.deg }} degrees</div>
        </div>
        <div>
          <h4 class="text-2xl pb-4">Humidity</h4>
          <div class="text-md">{{ conditions.humidity }}%</div>
        </div>
      </div>
    </div>
    <div
      class="flex items-center justify-center w-full md:self-end pb-10 md:pb-0"
    >
      <div
        @click.prevent="scrollView"
        class="border-2 rounded-full p-1.5 bg-gray-100 hover:cursor-pointer hover:bg-gray-200 transition-background-color"
      >
        <div
          i-carbon-arrow-down
          class="text-2xl bg-gray-500 hover:cursor-pointer"
        ></div>
      </div>
    </div>
  </div>
  <div
    id="i"
    class="flex flex-col items-center justify-between w-full p-10 bg-light-400"
  >
    <h2 class="text-3xl pb-5">Search Weather By City</h2>
    <input
      class="w-60 text-center text-2xl p-2"
      :class="[error ? 'bg-red-200' : '']"
      v-model="city"
      type="text"
    />

    <button
      class="w-60 p-2 bg-indigo-900 text-light-200 text-xl"
      @click.prevent="getLoc(city)"
    >
      Check Weather
    </button>
    <div v-if="error">No city with that name</div>
  </div>
</template>

<script setup lang="ts">
const city = ref('Toronto')
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
const cityName = ref('Toronto')
const error = ref(false)
const getLat = async (city: string) => {
  try {
    const res = await fetch(
      `https://api.openweathermap.org/geo/1.0/direct?q=${city},&limit=1&appid=5f5604ff8a6d663ad62ad8ac5d3ba2c1`
    )
    const data = res.json()
    return data
  } catch (err) {}
}
const getWeather = async (lat: string, lon: string) => {
  try {
    const res = await fetch(
      `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=5f5604ff8a6d663ad62ad8ac5d3ba2c1&units=metric`
    )
    const data = res.json()
    return data
  } catch (err) {}
}
const getLoc = async (location: string) => {
  error.value = false
  try {
    const data = await getLat(location)
    try {
      const weather = await getWeather(data[0].lat, data[0].lon)
      conditions.weather = weather.weather[0].main
      conditions.sky = weather.weather[0].description
      conditions.wind = weather.wind
      conditions.temp = weather.main.temp
      conditions.humidity = weather.main.humidity
      conditions.clouds = weather.clouds.all
      cityName.value = city.value
    } catch (err) {
      error.value = true
    }
  } catch (err) {
    error.value = true
  }
}
getLoc('Toronto')

const scrollView = () => {
  const element = document.querySelector('#i')
  if (element) element.scrollIntoView({ behavior: 'smooth' })
}
</script>

<style scoped></style>
