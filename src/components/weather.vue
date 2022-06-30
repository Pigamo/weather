<template>
  <!-- <div
    class="absolute w-full h-full transition-opacity bg-blue-900 -z-1"
    :class="[conditions.weather == 'Clouds' ? 'change' : '']"
  ></div> -->
  <div
    class="flex flex-wrap items-center transition-colors transition-background min-h-screen md:h-screen md:min-h-screen-sm md:p-8 px-8 py-4 justify-around"
    :style="[
      `transition: background 2s; background: linear-gradient(to bottom,  rgba(200,200,200,1), rgba(255,255,255,0.5) ${conditions.clouds}%), url(./mountains.svg) no-repeat bottom; background-size:cover`,
    ]"
  >
    <div class="md:text-8xl text-5xl md:p-8 pt-8">
      {{ city.charAt(0).toUpperCase() + city.slice(1) }}
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
        <!-- <div>
                <h4 class="text-2xl p-4">Temperature</h4>
          <div class="text-lg">{{ conditions.temp.toFixed() }}C</div>
        </div> -->
        <div>
          <div class="text-2xl pb-4">Wind</div>
          <div class="text-md">{{ conditions.wind.speed }} km/hr</div>
          <div class="text-md">{{ conditions.wind.deg }} degrees</div>
          <!-- <div class="p-4">{{ conditions.wind.gust }}</div> -->
        </div>
        <div>
          <h4 class="text-2xl pb-4">Humidity</h4>
          <div class="text-md">{{ conditions.humidity }}%</div>
        </div>
      </div>
    </div>
    <div class="flex items-center justify-center w-full self-end">
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
    <input class="w-60 text-center text-2xl p-2" v-model="city" type="text" />
    <button
      class="w-60 p-2 bg-indigo-900 text-light-200 text-xl"
      @click.prevent="getLoc(city)"
    >
      Check Weather
    </button>
  </div>
</template>

<script setup lang="ts">
const key = import.meta.env.API_KEY
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

const getLat = async (city: string) => {
  try {
    const res = await fetch(
      `https://api.openweathermap.org/geo/1.0/direct?q=${city},&limit=1&appid=${key}`
    )
    const data = res.json()
    return data
  } catch (err) {
    console.log(err)
  }
}
const getWeather = async (lat: string, lon: string) => {
  console.log(lat, lon)
  try {
    const res = await fetch(
      `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${key}&units=metric`
    )
    const data = res.json()
    return data
  } catch (err) {
    console.log(err)
  }
}
const getLoc = async (location: string) => {
  const data = await getLat(location)

  console.log('~~~~~~~~~~~~~~~~~~~~~~~\nDATA: ', data[0])
  console.log('lat: ', data[0].lat, '\nlong: ', data[0].lon)
  const weather = await getWeather(data[0].lat, data[0].lon)
  console.log(weather)
  console.log(weather.weather[0].main)
  conditions.weather = weather.weather[0].main
  conditions.sky = weather.weather[0].description
  conditions.wind = weather.wind
  conditions.temp = weather.main.temp
  conditions.humidity = weather.main.humidity
  conditions.clouds = weather.clouds.all
}
getLoc('Toronto')

const scrollView = () => {
  const element = document.querySelector('#i')
  if (element) element.scrollIntoView({ behavior: 'smooth' })
}
</script>

<style scoped>
@keyframes wipe {
  0% {
    opacity: 1;
  }
  25% {
    opacity: 0.25;
  }
  50% {
    opacity: 0;
  }
  75% {
    opacity: 0.25;
  }
  100% {
    opacity: 1;
  }
}
.change {
  animation: change 1s linear once;
}
</style>
