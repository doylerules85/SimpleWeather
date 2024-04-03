<script setup lang="ts">
import dayjs from "dayjs";
import customParseFormat from "dayjs/plugin/customParseFormat";
import { onMounted, ref } from "vue";

dayjs.extend(customParseFormat);

const cityTabs = [
  { city: "Rio de Janeiro" },
  { city: "Beijing" },
  { city: "Los Angeles" },
];
const loadingWeather = ref(false);

const selectedCity = ref<string | null>("Los Angeles");

const selectCity = (city: string) => {
  selectedCity.value = city;
  getForecastWeather();
  getHours();
};

const search = ref("");
const searchCity = () => {
  selectedCity.value = search.value;
  getForecastWeather();
  getHours();
  search.value = "";
};

let hourData = ref(null);
const getHours = async () => {
  loadingWeather.value = true;
  try {
    const response = await fetch(
      `https://api.openweathermap.org/data/2.5/forecast?q=${
        selectedCity.value
      }&cnt=12&units=imperial&appid=${import.meta.env.VITE_WEATHER_KEY}`
    );
    const data = await response.json();
    console.log(data);
    loadingWeather.value = false;
    hourData.value = data;
  } catch (err) {
    console.log(err);
  }
};

// for time - would create type here for TS
let forecastData = ref<null | { list: [any] }>(null);
const getForecastWeather = async () => {
  loadingWeather.value = true;
  try {
    const response = await fetch(
      `https://api.openweathermap.org/data/2.5/forecast?q=${
        selectedCity.value
      }&units=imperial&appid=${import.meta.env.VITE_WEATHER_KEY}`
    );
    const data = await response.json();
    console.log(data);
    loadingWeather.value = false;
    forecastData.value = data;
  } catch (err) {
    console.log(err);
  }
};

const refreshData = () => {
  getForecastWeather();
};

onMounted(() => {
  getForecastWeather();
  getHours();
});
</script>

<template>
  <main>
    <header>
      <h1>Simple Weather</h1>
      <form @submit.prevent="searchCity">
        <input
          type="text"
          placeholder="Search City"
          class="search-input"
          v-model="search"
        />
        <button class="btn btn--ghost">Submit</button>
      </form>
    </header>
    <section>
      <div role="tablist" class="tabs" aria-label="select city tabs">
        <button
          v-for="tab in cityTabs"
          role="tab"
          :aria-selected="tab.city === selectedCity ?? false"
          :class="{ selected: tab.city === selectedCity }"
          @click="selectCity(tab.city)"
        >
          {{ tab.city }}
        </button>
      </div>
      <div class="container refresh">
        <p class="city-label">Now Showing: {{ selectedCity }}</p>
        <button class="btn" @click="refreshData">Refresh</button>
      </div>
      <div class="loader" v-if="loadingWeather">
        <span>LOADING...</span>
      </div>
      <template v-else>
        <div class="container">
          <h2>Next Hours</h2>
          <div class="weather-grid">
            <div
              v-for="day in forecastData?.list"
              :key="day.dt"
              class="weather-box"
            >
              <span class="temp">{{ Math.floor(day.main.temp) }}°</span>
              <img
                :src="`https://openweathermap.org/img/wn/${day.weather[0].icon}.png`"
                alt=""
              />
              <span class="date">{{
                dayjs(day?.dt_txt).format("h:mm A")
              }}</span>
              <span class="rain">{{ Math.floor(day.pop * 100) }}%</span>
              <span class="desc">{{ day.weather[0].description }}</span>
            </div>
          </div>
        </div>
        <div class="container">
          <h2>Next 5 days</h2>
          <div class="weather-grid">
            <div
              v-for="day in forecastData?.list"
              :key="day.dt"
              class="weather-box"
            >
              <span class="temp">{{ Math.floor(day.main.temp) }}°</span>
              <img
                :src="`https://openweathermap.org/img/wn/${day.weather[0].icon}.png`"
                alt=""
              />
              <span class="date">{{
                dayjs(day?.dt_txt).format("ddd,MMM D")
              }}</span>
              <span class="rain">{{ Math.floor(day.pop * 100) }}%</span>
              <span class="desc">{{ day.weather[0].description }}</span>
            </div>
          </div>
        </div>
      </template>
    </section>
  </main>
</template>

<style scoped></style>
