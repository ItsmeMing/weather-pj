<!-- Use preprocessors via the lang attribute! e.g. <template lang="pug"> -->
<template>
  <div id="app">
    <section class="wrapper" ref="wrapper">
      <div class="container">
        <header>
          <div class="header__location">
            <img src="./components/icons/location.png" />
            <div>
              <p :ref="latitude">{{ latitude }}</p>
              <img src="./components/icons/dropdown.png" />
            </div>
          </div>
          <div class="header__bell">
            <img src="./components/icons/bell.png" />
          </div>
        </header>
        <main>
          <div class="todays-weather">
            <img src="./components/icons/sun-cloud-angled-rain.png" />
            <p class="todays-weather__temp">
              {{ temp_hour + temp_unit }}
            </p>
            <p class="todays-weather__precipitations">Precipitations</p>
            <div class="todays-weather__minmax">
              <p id="max">Max.: {{ minTemp + temp_unit }}</p>
              <p id="min">Min.: {{ maxTemp + temp_unit }}</p>
            </div>
          </div>
          <div class="location-info">
            <div class="location-info__rain">
              <img src="./components/icons/liquid.png" />
              <p>
                {{ rain_hour + rain_unit }}
              </p>
            </div>
            <div class="location-info__humidity">
              <img src="./components/icons/humidity.png" />
              <p>
                {{ humidity_hour + humidity_unit }}
              </p>
            </div>
            <div class="location-info__wind">
              <img src="./components/icons/wind.png" />
              <p>
                {{ wind_hour + wind_unit }}
              </p>
            </div>
          </div>
          <section class="forecasts">
            <div class="today-forecast">
              <div class="today">
                <h1>{{ day_name_obj[dateObj.getUTCDay()] }}</h1>
                <p class="todays__date">
                  {{
                    `${
                      month_name_obj[dateObj.getMonth()]
                    }, ${dateObj.getDate()}`
                  }}
                </p>
              </div>
              <div class="forecast">
                <ul>
                  <li v-for="(tData, key) in todayWeatherData" :key="key">
                    <p class="forecast__hour">{{ tData.temp }}</p>
                    <img src="./components/icons/cloud-with-sun.png" />
                    <p class="forecast__hour">{{ tData.time.slice(-5) }}</p>
                  </li>
                </ul>
              </div>
            </div>
            <div class="nextdays-forecast">
              <div class="nextdays-forecast__header">
                <h1>Next Forecast</h1>
                <img src="./components/icons/calendar.png" />
              </div>
              <div class="nextdays">
                <div
                  class="nextday"
                  v-for="(nextday, key) in nextdaysWeatherData"
                  :key="key"
                >
                  <p>
                    {{
                      new Date(nextday.date).toLocaleString("en-us", {
                        weekday: "long",
                      })
                    }}
                  </p>
                  <img src="./components/icons/lightning.png" />
                  <div>
                    <p class="nextday__min">
                      {{ nextday.minTemp }}<sup>{{ temp_unit }}</sup>
                    </p>
                    <p class="nextday__max">
                      {{ nextday.maxTemp }}<sup>{{ temp_unit }}</sup>
                    </p>
                  </div>
                </div>
              </div>
            </div>
          </section>
        </main>
      </div>
    </section>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
const wrapper = ref(null);
const data = ref(null);
const url = ref(
  "https://api.open-meteo.com/v1/forecast?latitude=10.80&longitude=106.64&hourly=temperature_2m,relativehumidity_2m,rain,windspeed_10m&daily=temperature_2m_max,temperature_2m_min&timezone=Asia%2FBangkok"
);
const latitude = ref(null);
const dateObj = ref(new Date());

//weather units
const temp_unit = ref(null);
const rain_unit = ref(null);
const humidity_unit = ref(null);
const wind_unit = ref(null);

//weather data
const time_current = ref(null);
const forecastWeatherData = ref([]);
let todayWeatherData;
let nextdaysWeatherData = ref([]);
let minTemp;
let maxTemp;
let temp_hour;
let rain_hour;
let humidity_hour;
let wind_hour;

//day object
const day_name_obj = ref({
  0: "Sunday",
  1: "Monday",
  2: "Tuesday",
  3: "Wednesday",
  4: "Thursday",
  5: "Friday",
  6: "Saturday",
});

//month object
const month_name_obj = ref({
  0: "January",
  1: "February",
  2: "March",
  3: "April",
  4: "May",
  5: "June",
  6: "July",
  7: "August",
  8: "September",
  9: "October",
  10: "November",
  11: "December",
});

onMounted(async () => {
  data.value = await (await fetch(url.value)).json();
  latitude.value = data.value.latitude;
  for (let i = 0; i < data.value.daily.time.length; i++) {
    nextdaysWeatherData.value.push({
      date: data.value.daily.time[i],
      minTemp: data.value.daily.temperature_2m_min[i],
      maxTemp: data.value.daily.temperature_2m_max[i],
    });
  }

  if (dateObj.value.getHours() >= 6 && dateObj.value.getHours() < 18) {
    wrapper.value.style.background =
      "linear-gradient(167.44deg, #29B2DD 0%, #33AADD 47.38%, #2DC8EA 100%)";
  } else {
    wrapper.value.style.background =
      "linear-gradient(167.44deg, #08244F 0%, #134CB5 47.38%, #0B42AB 100%)";
  }

  //get weather units
  temp_unit.value = data.value.hourly_units.temperature_2m;
  rain_unit.value = data.value.hourly_units.rain;
  humidity_unit.value = data.value.hourly_units.relativehumidity_2m;
  wind_unit.value = data.value.hourly_units.windspeed_10m;

  const month = (dateObj.value.getMonth() + 1).toString().padStart(2, "0");
  const day = dateObj.value.getDate().toString().padStart(2, "0");
  time_current.value = `${month}-${day}`;

  for (let i = 0; i < data.value.hourly.time.length; i++) {
    forecastWeatherData.value.push({
      time: data.value.hourly.time[i],
      temp: data.value.hourly.temperature_2m[i],
      rain: data.value.hourly.rain[i],
      humidity: data.value.hourly.relativehumidity_2m[i],
      wind: data.value.hourly.windspeed_10m[i],
    });
  }

  todayWeatherData = computed(() =>
    forecastWeatherData.value.filter((data) =>
      data.time.includes(time_current.value)
    )
  );

  minTemp = computed(() =>
    Math.min(...todayWeatherData.value.map((tData) => tData.temp))
  );
  maxTemp = computed(() =>
    Math.max(...todayWeatherData.value.map((tData) => tData.temp))
  );
  temp_hour = computed(
    () => todayWeatherData.value[dateObj.value.getHours()].temp
  );
  rain_hour = computed(
    () => todayWeatherData.value[dateObj.value.getHours()].rain
  );
  humidity_hour = computed(
    () => todayWeatherData.value[dateObj.value.getHours()].humidity
  );
  wind_hour = computed(
    () => todayWeatherData.value[dateObj.value.getHours()].wind
  );
  console.log(todayWeatherData.value);
});
</script>

<!-- Use preprocessors via the lang attribute! e.g. <style lang="scss"> -->
<style lang="scss">
sup {
  vertical-align: super !important;
  font-size: 12px !important;
}
#app {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100vw;
  background-color: black;
  .wrapper {
    width: 423px;
    height: 858px;
    border-radius: 40px;
    .container {
      width: 80%;
      height: calc(100% - 90px);
      margin: 45px auto 0 auto;
      header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        .header__location {
          display: flex;
          align-items: center;
          gap: 15px;
          div {
            display: flex;
            align-items: center;
            gap: 20px;
            cursor: pointer;
          }
        }
      }
      main {
        .todays-weather {
          text-align: center;
          img {
            filter: drop-shadow(2px 4px 30px rgba(0, 0, 0, 0.05));
          }
          .todays-weather__temp {
            font-weight: 600;
            font-size: 64px;
            line-height: 76px;
          }
          .todays-weather__precipitations,
          .todays-weather__minmax {
            font-size: 18px;
            line-height: 21px;
          }
          .todays-weather__minmax {
            display: flex;
            justify-content: center;
            gap: 10px;
          }
        }
        .location-info {
          display: flex;
          margin-top: 45px;
          padding: 15px 0;
          background: rgba(0, 16, 38, 0.3);
          box-shadow: 0px 0px 20px rgba(0, 0, 0, 0.25);
          border-radius: 20px;
          .location-info__rain,
          .location-info__humidity,
          .location-info__wind {
            flex: 0 0 33%;
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 10px;
            font-size: 14px;
          }
        }
        .forecasts {
          height: 275px;
          margin-top: 20px;
          overflow-y: scroll;
          .today-forecast {
            font-size: 18px;
            margin-bottom: 20px;
            padding: 15px 0;
            background: rgba(0, 16, 38, 0.3);
            box-shadow: 0px 0px 20px rgba(0, 0, 0, 0.25);
            border-radius: 20px;
            .today,
            .forecast {
              width: calc(100% - 36px);
            }
            .today {
              display: flex;
              justify-content: space-between;
              margin: 0 auto 12px auto;
              h1 {
                font-weight: 700;
                font-size: 20px;
              }
            }
            .forecast {
              overflow: auto;
              margin: 0 auto;
              ul {
                display: flex;
                gap: 12px;
                li {
                  text-align: center;
                  padding: 10px;
                  img {
                    margin: 30px 0;
                  }
                }
              }
            }
          }
          .nextdays-forecast {
            background: rgba(0, 16, 38, 0.3);
            box-shadow: 0px 0px 20px rgba(0, 0, 0, 0.25);
            border-radius: 20px;
            .nextdays-forecast__header,
            .nextdays {
              width: calc(100% - 36px);
              margin: 0 auto;
              padding-bottom: 10px;
            }
            .nextdays-forecast__header {
              display: flex;
              justify-content: space-between;
              padding: 20px 0;
              h1 {
                font-weight: 700;
                font-size: 20px;
              }
              img {
              }
            }
            .nextdays {
              .nextday {
                display: flex;
                align-items: center;
                justify-content: space-between;
                margin-bottom: 10px;
                p {
                  flex: 1;
                }
                img {
                  width: 50px;
                  height: auto;
                }
                div {
                  flex: 1;
                  display: flex;
                  text-align: right;
                  .nextday__min {
                  }
                  .nextday__max {
                  }
                }
              }
            }
          }
        }
        ::-webkit-scrollbar {
          display: none;
        }
      }
    }
  }
}
</style>
