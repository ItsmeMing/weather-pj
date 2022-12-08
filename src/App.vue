<!-- Use preprocessors via the lang attribute! e.g. <template lang="pug"> -->
<template>
    <div id="app">
        <section class="wrapper light" ref="wrapper">
            <div class="container">
                <header>
                    <div class="header__location">
                        <img src="./components/icons/location.png" />
                        <div class="coordinates" @click="() => form.classList.toggle('active')">
                            <p v-if="data">{{ data.timezone }}</p>
                            <img src="./components/icons/dropdown.png" />
                        </div>
                        <form class="coordinates-form" ref="form">
                            <div class="latitude">
                                <label>Latitude</label>
                                <input type="number" v-model="latitude" />
                            </div>
                            <div class="longitude">
                                <label>longitude</label>
                                <input type="number" v-model="longitude" />
                            </div>
                            <button @click.prevent="handleInput(latitude, longitude)">Find</button>
                            <p class="err-mess" :ref="err">{{ err }}</p>
                        </form>
                    </div>
                    <div class="header__bell">
                        <img src="./components/icons/bell.png" />
                    </div>
                </header>
                <main>
                    <div class="clicked-date">
                        <img :src="imgSrc" />
                        <p class="clicked-date__temp">
                            {{ currentWeatherInfo.temp_hour + temp_unit }}
                        </p>
                        <p class="clicked-date__precipitations">{{ currentWeatherInfo.time }}</p>
                        <p class="clicked-date__precipitations">Precipitations</p>
                        <div class="clicked-date__minmax">
                            <p id="max">Max.: {{ currentWeatherInfo.maxTemp + temp_unit }}</p>
                            <p id="min">Min.: {{ currentWeatherInfo.minTemp + temp_unit }}</p>
                        </div>
                    </div>
                    <div class="location-info">
                        <div class="location-info__rain">
                            <img src="./components/icons/liquid.png" />
                            <p>
                                {{ currentWeatherInfo.rain_hour }}
                                <span>{{ rain_unit }}</span>
                            </p>
                        </div>
                        <div class="location-info__humidity">
                            <img src="./components/icons/humidity.png" />
                            <p>
                                {{ currentWeatherInfo.humidity_hour }}
                                <span>{{ humidity_unit }}</span>
                            </p>
                        </div>
                        <div class="location-info__wind">
                            <img src="./components/icons/wind.png" />
                            <p>
                                {{ currentWeatherInfo.wind_hour }}
                                <span>{{ wind_unit }}</span>
                            </p>
                        </div>
                    </div>
                    <section class="forecasts">
                        <div class="today-forecast">
                            <div class="today">
                                <h1>{{ dayName }}</h1>
                                <p class="clicked__date">
                                    {{ `${monthName}, ${parseInt(currentDay, 10)}` }}
                                </p>
                            </div>
                            <div class="forecast">
                                <ul>
                                    <li
                                        v-for="(tData, key) in clickedDayWeatherData"
                                        :key="key"
                                        @click="handleShowDataByHour(key)"
                                    >
                                        <p class="forecast__hour">
                                            {{ tData.temp + temp_unit }}
                                        </p>
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
                                    v-for="(nextday, key) in weatherDataByDay"
                                    :key="key"
                                    @click="handleShowDataByDay(nextday.date)"
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
                                            {{ nextday.minTemp + temp_unit }}
                                        </p>
                                        <p class="nextday__max">
                                            {{ nextday.maxTemp + temp_unit }}
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
import {ref, computed, onMounted, watchEffect, watch} from "vue";
import suncloud from "./components/icons/sun-cloud-angled-rain.png";
import sunrain from "./components/icons/sun-cloud-rain.png";

//big img src
const imgSrc = ref("");

//form ref
const form = ref();

//err mess
const err = ref(null);

//lat and long
const latitude = ref();
const longitude = ref();
const url = ref(
    "https://api.open-meteo.com/v1/forecast?latitude=48.85&longitude=2.35&hourly=temperature_2m,relativehumidity_2m,rain,windspeed_10m&daily=temperature_2m_max,temperature_2m_min&timezone=auto",
);

const wrapper = ref(null);
const data = ref(null);
const dateObj = ref(new Date());
const newDateObj = () => (dateObj.value = new Date());

//weather units
const temp_unit = ref(null);
const rain_unit = ref(null);
const humidity_unit = ref(null);
const wind_unit = ref(null);

//date
const locationTime = ref("");

watch(
    () => dateObj.value,
    () => {
        locationTime.value = dateObj.value
            .toLocaleString("en-US", {
                hour: "2-digit",
                minute: "2-digit",
                second: "2-digit",
                day: "2-digit",
                month: "2-digit",
                year: "numeric",
                hour12: false,
                timeZone: data.value.timezone,
            })
            .replace(/\//g, "-");
        if (locationTime.value.slice(12, 20) === "00:00:00") {
            url.value = `https://api.open-meteo.com/v1/forecast?latitude=${latitude.value}&longitude=${longitude.value}&hourly=temperature_2m,relativehumidity_2m,rain,windspeed_10m&daily=temperature_2m_max,temperature_2m_min&timezone=auto`;
        }
    },
);

const dayName = computed(() => {
    return new Date(locationTime.value.slice(0, 10)).toLocaleString("en-us", {
        weekday: "long",
    });
});
const monthName = computed(() => {
    return new Date(locationTime.value.slice(0, 10)).toLocaleString("en-us", {
        month: "short",
    });
});
const currentDate = computed(() => {
    return locationTime.value.slice(0, 5);
});
const currentDay = computed(() => {
    return locationTime.value.slice(3, 5);
});
const currentHour = computed(() => {
    return locationTime.value.slice(12, 14);
});

//weather data
const weatherDataByHour = ref([]);
const weatherDataByDay = ref([]);
const clickedDayWeatherData = ref([]);
const currentWeatherInfo = ref({
    time: undefined,
    minTemp: undefined,
    maxTemp: undefined,
    temp_hour: undefined,
    rain_hour: undefined,
    humidity_hour: undefined,
    wind_hour: undefined,
});

//close form
const handleCloseForm = () => {
    form.value.classList.remove("active");
    err.value = "";
    latitude.value = "";
    longitude.value = "";
};

const handleInput = (lat, long) => {
    const newLat = parseFloat(lat).toFixed(2);
    const newLong = parseFloat(long).toFixed(2);
    if (newLat < -90 || newLat > 90 || isNaN(newLat)) {
        err.value = "Invalid Latitude!";
    } else if (newLong < -180 || newLong > 180 || isNaN(newLong)) {
        err.value = "Invalid longitude!";
    } else {
        handleCloseForm();
        setTimeout(() => {
            form.value.classList.remove("active");
        }, 0);
        url.value = `https://api.open-meteo.com/v1/forecast?latitude=${newLat}&longitude=${newLong}&hourly=temperature_2m,relativehumidity_2m,rain,windspeed_10m&daily=temperature_2m_max,temperature_2m_min&timezone=auto`;
    }
};

//change background color based on hour
const handleBackground = (hour) => {
    if (hour >= 6 && hour < 18) {
        imgSrc.value = suncloud;
        wrapper.value.classList.replace("dark", "light");
    } else {
        imgSrc.value = sunrain;
        wrapper.value.classList.replace("light", "dark");
    }
};

const handleShowDataByHour = (hour) => {
    currentWeatherInfo.value.time =
        clickedDayWeatherData.value.length !== 0 ? clickedDayWeatherData.value[hour].time.replace("T", " ") : "NaN";
    currentWeatherInfo.value.minTemp = Math.min(...clickedDayWeatherData.value.map((tData) => tData.temp));
    currentWeatherInfo.value.maxTemp = Math.max(...clickedDayWeatherData.value.map((tData) => tData.temp));
    currentWeatherInfo.value.temp_hour =
        clickedDayWeatherData.value.length !== 0 ? clickedDayWeatherData.value[hour].temp : "NaN";
    currentWeatherInfo.value.rain_hour =
        clickedDayWeatherData.value.length !== 0 ? clickedDayWeatherData.value[hour].rain : "NaN";
    currentWeatherInfo.value.humidity_hour =
        clickedDayWeatherData.value.length !== 0 ? clickedDayWeatherData.value[hour].humidity : "NaN";
    currentWeatherInfo.value.wind_hour =
        clickedDayWeatherData.value.length !== 0 ? clickedDayWeatherData.value[hour].wind : "NaN";
    handleBackground(hour);
};

const handleShowDataByDay = (day) => {
    clickedDayWeatherData.value = weatherDataByHour.value.filter((data) => data.time.includes(day));
};

//get location's weather data
watchEffect(async () => {
    //get data
    data.value = await fetch(url.value).then((res) => res.json());

    //get weather units
    temp_unit.value = data.value.hourly_units.temperature_2m;
    rain_unit.value = data.value.hourly_units.rain;
    humidity_unit.value = data.value.hourly_units.relativehumidity_2m;
    wind_unit.value = data.value.hourly_units.windspeed_10m;

    //format data by hour
    weatherDataByHour.value = [];
    for (let i = 0; i < data.value.hourly.time.length; i++) {
        weatherDataByHour.value.push({
            time: data.value.hourly.time[i],
            temp: data.value.hourly.temperature_2m[i],
            rain: data.value.hourly.rain[i],
            humidity: data.value.hourly.relativehumidity_2m[i],
            wind: data.value.hourly.windspeed_10m[i],
        });
    }

    //format data by day
    weatherDataByDay.value = [];
    for (let i = 0; i < data.value.daily.time.length; i++) {
        weatherDataByDay.value.push({
            date: data.value.daily.time[i],
            minTemp: data.value.daily.temperature_2m_min[i],
            maxTemp: data.value.daily.temperature_2m_max[i],
        });
    }
    handleShowDataByDay(currentDate.value);
    handleShowDataByHour(parseInt(currentHour.value, 10));
});
onMounted(() => {
    //re-check every 1s
    setInterval(newDateObj, 1000);
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
    height: 100vh;
    background-color: black;
    .wrapper {
        width: 423px;
        border-radius: 40px;
        .container {
            header {
                position: relative;
                z-index: 2;
                display: flex;
                justify-content: space-between;
                align-items: center;
                width: 80%;
                margin: 45px auto 0 auto;
                .header__location {
                    display: flex;
                    align-items: center;
                    gap: 15px;
                    .coordinates {
                        position: relative;
                        display: flex;
                        align-items: center;
                        gap: 20px;
                        cursor: pointer;
                        &:hover {
                            text-decoration: underline;
                        }
                    }
                    .coordinates-form {
                        display: none;
                        position: absolute;
                        top: 200%;
                        width: 200px;
                        padding: 15px 10px;
                        background: rgba(0, 16, 38, 0.4);
                        box-shadow: 0px 0px 20px rgba(0, 0, 0, 0.25);
                        border-radius: 20px;
                        div,
                        button {
                            width: 90%;
                            margin: 10px auto;
                        }
                        div {
                            display: flex;
                            justify-content: space-between;
                            align-items: center;
                            label,
                            input {
                                width: 50%;
                            }
                            input {
                                color: black;
                            }
                        }
                        button {
                            display: block;
                            font-weight: bold;
                            color: black;
                            padding: 5px 0;
                            background-color: rgba(100%, 100%, 100%, 60%);
                            backdrop-filter: blur(50px);
                            border: none;
                            border-radius: 10px;
                            cursor: pointer;
                        }
                        .err-mess {
                            text-align: center;
                            font-weight: bold;
                            color: red;
                        }
                    }
                    .coordinates-form.active {
                        display: block;
                    }
                }
            }
            main {
                position: relative;
                z-index: 1;
                > *:not(:last-child) {
                    width: 80%;
                    margin: 0 auto;
                }
                .clicked-date {
                    text-align: center;
                    img {
                        width: 344px;
                        height: 247px;
                        object-fit: cover;
                        filter: drop-shadow(2px 4px 30px rgba(0, 0, 0, 0.05));
                    }
                    .clicked-date__temp {
                        font-weight: 600;
                        font-size: 64px;
                        line-height: 76px;
                    }
                    .clicked-date__precipitations,
                    .clicked-date__minmax {
                        font-size: 18px;
                        line-height: 21px;
                    }
                    .clicked-date__minmax {
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
                    > * {
                        width: 80%;
                        margin: 0 auto;
                    }
                    .today-forecast {
                        font-size: 18px;
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
                                gap: 10px;
                                li {
                                    flex-basis: 25%;
                                    text-align: center;
                                    font-size: 17.5px;
                                    padding: 10px;
                                    &:hover {
                                        background: rgba(37, 102, 163, 0.2);
                                        border-radius: 20px;
                                    }
                                    img {
                                        margin: 30px 0;
                                    }
                                }
                            }
                        }
                    }
                    .nextdays-forecast {
                        margin: 20px auto;
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
                        }
                        .nextdays {
                            .nextday {
                                display: flex;
                                align-items: center;
                                justify-content: space-between;
                                margin-bottom: 10px;
                                &:hover {
                                    background: rgba(37, 102, 163, 0.2);
                                    border-radius: 10px;
                                }
                                p {
                                    flex: 1;
                                }
                                img {
                                    width: 40px;
                                    height: auto;
                                }
                                div {
                                    flex: 1;
                                    display: flex;
                                    justify-content: center;
                                    text-align: right;
                                    .nextday__min,
                                    .nextday__max {
                                        display: block;
                                        width: fit-content;
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
    .wrapper.light {
        background: linear-gradient(167.44deg, #29b2dd 0%, #33aadd 47.38%, #2dc8ea 100%);
    }
    .wrapper.dark {
        background: linear-gradient(167.44deg, #08244f 0%, #134cb5 47.38%, #0b42ab 100%);
    }
}
</style>
