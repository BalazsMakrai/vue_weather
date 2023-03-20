<template>
  <div class="flex flex-col flex-1 items-center">
    <div v-if="route.query.preview" class="text-white p-4 bg-weather-secondary w-full text-center">
      <p>
        You are currently previewing this city. Click the "+" icon to start
        tracking this location.
      </p>
    </div>
    <!-- Weather overview-->
    <div class="flex flex-col items-center text-white py-12">
      <h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
      <p class="text-sm mb-12">
        {{
          new Date(weatherData.currentTime).toLocaleDateString("en-us", {
            weekday: "long",
            day: "2-digit",
            month: "long",
          })
        }}
        {{
          new Date(weatherData.currentTime).toLocaleTimeString("hu-hu", {
            timeStyle: "short",
          })
        }}
      </p>
      <p class="text-8xl mb-8">
        {{ Math.round(weatherData.current.temp) }} &#8451;
      </p>
      <p>Feels like {{ Math.round(weatherData.current.feels_like) }} &#8451;</p>
      <p class="capitalize">{{ weatherData.current.weather[0].description }}</p>
      <img class="w-[150px] h-auto"
        :src="`http://openweathermap.org/img/wn/${weatherData.current.weather[0].icon}@2x.png`" alt="" />
    </div>
    <hr class="border-white border-opacity-10 border w-full" />

    <!-- Hourly forecast-->
    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">Hourly Weather</h2>
        <div class="flex gap-10 overflow-x-auto hover:overflow-x-scroll pb-5">
          <div v-for="hourData in weatherData.hourly" :key="hourData.dt"
            class="flex flex-col gap-4 items-center min-w-max">
            <p class="whitespace-nowrap text-md">
              {{
                new Date(hourData.currentTime).toLocaleTimeString("hu-hu", {
                  hour: "2-digit",
                })
              }}
            </p>
            <img class="w-auto h-[50px] object-cover"
              :src="`http://openweathermap.org/img/wn/${hourData.weather[0].icon}@2x.png`" alt="" />
            <p class="text-xl">{{ Math.round(hourData.temp) }} &#8451;</p>
          </div>
        </div>
      </div>
    </div>
    <hr class="border-white border-opacity-10 border w-full" />

    <!-- Weekly forecast-->
    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">7 Day Forecast</h2>
        <div v-for="day in weatherData.daily" :key="day.dt" class="flex items-center border-b border-indigo-200">
          <p class="flex-1">
            {{
              new Date(day.dt * 1000).toLocaleDateString("en-us", {
                weekday: "long",
                day: "2-digit"
              })
            }}
          </p>
          <img class="w-[50px] h-[50px] object-cover"
            :src="`http://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png`" alt="" />
          <div class="flex gap-2 flex-1 justify-end">
            <p>Low: {{ Math.round(day.temp.min) }}</p>
            <p>High: {{ Math.round(day.temp.max) }}</p>
          </div>
          <div class="flex gap-2 flex-1 justify-end">
            <p>Sunrise: {{
              new Date(day.sunrise * 1000).toLocaleTimeString("hu-hu", {
                hour: "2-digit",
                minute: "2-digit"
              })
            }} </p>
          </div>
          <div class="flex gap-2 flex-1 justify-end">
            <p>Sunset: {{
              new Date(day.sunset * 1000).toLocaleTimeString("hu-hu", {
                hour: "2-digit",
                minute: "2-digit"
              })
            }} </p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import { useRoute } from "vue-router";

const route = useRoute();
const weatherAPIKey = import.meta.env.VITE_WEATHER_API;
const getWeatherData = async () => {
  try {
    const weatherData = await axios.get(
      `https://api.openweathermap.org/data/2.5/onecall?lat=${route.query.lat}&lon=${route.query.lng}&appid=${weatherAPIKey}&units=metric`
    );
    const localOffset = new Date().getTimezoneOffset() * 60000;
    const utc = weatherData.data.current.dt * 1000 + localOffset;
    weatherData.data.currentTime =
      utc + 1000 * weatherData.data.timezone_offset;

    weatherData.data.hourly.forEach((hour) => {
      const utc = hour.dt * 1000 + localOffset;
      hour.currentTime = utc + 1000 * weatherData.data.timezone_offset;
    });
    return weatherData.data;
  } catch (err) {
    console.log(err);
  }
};
const weatherData = await getWeatherData();
console.log(weatherData.daily);
</script>
