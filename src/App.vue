<!-- Use preprocessors via the lang attribute! e.g. <template lang="pug"> -->
<template>
    <div id="app">
        <section class="wrapper light" ref="wrapper">
            <div class="container">
                <header>
                    <div class="header__location">
                        <img src="./assets/icons/location.png" />
                        <div class="coordinates" @click="() => form.classList.toggle('active')">
                            <p>{{ currentLocation }}</p>
                            <img src="./assets/icons/dropdown.png" />
                        </div>
                        <form class="coordinates-form" ref="form">
                            <div class="latitude">
                                <label for="latitude">Latitude</label>
                                <input id="latitude" type="number" v-model="latitudeInput" />
                            </div>
                            <div class="longitude">
                                <label for="longitude">Longitude</label>
                                <input id="longitude" type="number" v-model="longitudeInput" />
                            </div>
                            <button @click.prevent="handleInput(latitudeInput, longitudeInput)">Find</button>
                            <p class="err-mess" :ref="err">{{ err }}</p>
                        </form>
                    </div>
                    <div class="header__bell">
                        <img src="./assets/icons/bell.png" />
                    </div>
                </header>
                <main>
                    <div class="clicked-date">
                        <img :src="hourIcon" />
                        <p class="clicked-date__temp">
                            {{ clickedHourWeatherData.temp + weatherUnits.temp_unit }}
                        </p>
                        <p class="clicked-date__precipitations">
                            {{ clickedHourWeatherData?.time?.replace("T", ", ") }}
                        </p>
                        <p class="clicked-date__precipitations">Precipitations</p>
                        <div class="clicked-date__minmax">
                            <p id="max">
                                Max.:
                                {{
                                    Math.max(...clickedDayWeatherData.map((hData) => hData.temp)) +
                                    weatherUnits.temp_unit
                                }}
                            </p>
                            <p id="min">
                                Min.:
                                {{
                                    Math.min(...clickedDayWeatherData.map((hData) => hData.temp)) +
                                    weatherUnits.temp_unit
                                }}
                            </p>
                        </div>
                    </div>
                    <div class="location-info">
                        <div class="location-info__precipitation">
                            <img src="./assets/icons/liquid.png" />
                            <p>
                                {{ clickedHourWeatherData.precipitation + weatherUnits.precipitation_unit }}
                            </p>
                        </div>
                        <div class="location-info__humidity">
                            <img src="./assets/icons/humidity.png" />
                            <p>
                                {{ clickedHourWeatherData.humidity + weatherUnits.humidity_unit }}
                            </p>
                        </div>
                        <div class="location-info__wind">
                            <img src="./assets/icons/wind.png" />
                            <p>
                                {{ clickedHourWeatherData.wind + weatherUnits.wind_unit }}
                            </p>
                        </div>
                    </div>
                    <section class="forecasts">
                        <div class="today-forecast">
                            <div class="today">
                                <h1>
                                    {{
                                        new Date(clickedDayWeatherData[0]?.time.slice(0, 10)).toLocaleString("en-CA", {
                                            weekday: "long",
                                        })
                                    }}
                                </h1>
                                <p class="clicked__date">
                                    {{
                                        new Date(clickedDayWeatherData[0]?.time.slice(0, 10)).toLocaleString("en-CA", {
                                            month: "short",
                                            day: "numeric",
                                        })
                                    }}
                                </p>
                            </div>
                            <div class="forecast">
                                <ul>
                                    <li
                                        v-for="(hData, key) in clickedDayWeatherData"
                                        :key="key"
                                        @click="handleHourWeatherData(key)"
                                    >
                                        <p class="forecast__hour">
                                            {{ hData.temp + weatherUnits.temp_unit }}
                                        </p>
                                        <figure><img :src="hData.icon" /></figure>
                                        <p class="forecast__hour">{{ hData.time.slice(-5) }}</p>
                                    </li>
                                </ul>
                            </div>
                        </div>
                        <div class="nextdays-forecast">
                            <div class="nextdays-forecast__header">
                                <h1>Next Forecast</h1>
                                <img src="./assets/icons/calendar.png" />
                            </div>
                            <div class="nextdays">
                                <section
                                    v-for="(data, key) in forecastWeatherData"
                                    :key="key"
                                    @click="handleDayWeatherData(key)"
                                >
                                    <div class="nextday">
                                        <p>
                                            {{
                                                new Date(data[0]?.time.slice(0, 10)).toLocaleString("en-CA", {
                                                    weekday: "long",
                                                })
                                            }}
                                        </p>
                                        <img
                                            :src="
                                                setWeatherIcon(
                                                    10,
                                                    Math.max(...data.map((d) => d.temp)),
                                                    Math.max(...data.map((d) => d.precipitation)),
                                                )
                                            "
                                        />
                                        <div>
                                            <p class="nextday__min">
                                                {{ Math.min(...data.map((d) => d.temp)) + weatherUnits.temp_unit }}
                                            </p>
                                            <p class="nextday__max">
                                                {{ Math.max(...data.map((d) => d.temp)) + weatherUnits.temp_unit }}
                                            </p>
                                        </div>
                                    </div>
                                </section>
                            </div>
                        </div>
                    </section>
                </main>
            </div>
        </section>
    </div>
</template>

<script setup>
import {computed, onBeforeMount, onBeforeUnmount, ref, watch} from "vue";
import cloudwithsun from "./assets/icons/cloud-with-sun.png";
import cloudwithmoon from "./assets/icons/cloud-with-moon.png";
import cloud from "./assets/icons/cloud.png";
import rain from "./assets/icons/rain.png";
import lightning from "./assets/icons/lightning.png";

const latitude = ref();
const longitude = ref();
const weatherUrl = computed(() => {
    return `https://api.open-meteo.com/v1/forecast?latitude=${latitude.value}&longitude=${longitude.value}&hourly=temperature_2m,relativehumidity_2m,precipitation,windspeed_10m&timezone=auto`;
});
const cityUrl = computed(() => {
    return `https://api.opencagedata.com/geocode/v1/json?key=0e5b1344d45b497d9a969f1dcaa8d35d&q=${latitude.value}+${longitude.value}`;
});
const locationTime = ref("");
const weatherUnits = ref({temp_unit: "", precipitation_unit: "", humidity_unit: "", wind_unit: ""});
const forecastWeatherData = ref({
    0: [],
    1: [],
    2: [],
    3: [],
    4: [],
    5: [],
    6: [],
});
const clickedDayWeatherData = ref([]);
const clickedHourWeatherData = ref({});

//big img's ref
const hourIcon = ref("");
//screen's ref
const wrapper = ref();
const dateObj = ref(new Date());
const currentLocation = ref(null);
const timeZone = ref("");
const latitudeInput = ref();
const longitudeInput = ref();
const form = ref();
const err = ref();
let timeout;

const refresh = () => (dateObj.value = new Date());
const interval = setInterval(refresh, 1000);

watch(
    () => dateObj.value,
    () => {
        locationTime.value = dateObj.value.toLocaleString("en-CA", {
            hour: "2-digit",
            minute: "2-digit",
            second: "2-digit",
            year: "numeric",
            month: "2-digit",
            day: "2-digit",
            hour12: false,
            timeZone: timeZone.value,
        });
        if (locationTime.value.slice(-8) === "00:00:00") getLocationsWeatherData();
        else if (locationTime.value.slice(-5) === "00:00")
            handleHourWeatherData(parseInt(locationTime.value.slice(12, 14), 10));
    },
);

//close form
const handleCloseForm = () => {
    form.value.classList.remove("active");
    err.value = "";
    latitudeInput.value = "";
    longitudeInput.value = "";
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
        latitude.value = newLat;
        longitude.value = newLong;
        getLocationsWeatherData();
        setTimeout(() => {
            form.value.classList.remove("active");
        }, 0);
    }
};

const changeBackgroundClr = (hour) => {
    if (hour >= 6 && hour <= 18) {
        wrapper.value.classList.replace("dark", "light");
    } else {
        wrapper.value.classList.replace("light", "dark");
    }
};

const setWeatherIcon = (hour, temp, precipitation) => {
    if (hour >= 6 && hour <= 18) {
        if (temp > 25) {
            if (precipitation >= 1.5 && precipitation <= 5) return rain;
            else if (precipitation > 5) return lightning;
            else return cloudwithsun;
        } else {
            if (precipitation >= 1.5 && precipitation <= 5) return rain;
            else if (precipitation > 5) return lightning;
            else return cloud;
        }
    } else {
        if (temp > 25) {
            if (precipitation >= 1.5 && precipitation <= 5) return rain;
            else if (precipitation > 5) return lightning;
            else return cloudwithmoon;
        } else {
            if (precipitation >= 1.5 && precipitation <= 5) return rain;
            else if (precipitation > 5) return lightning;
            else return cloud;
        }
    }
};

const handleDayWeatherData = (dIndex) => {
    clickedDayWeatherData.value = forecastWeatherData.value[dIndex];
};

const handleHourWeatherData = (hIndex) => {
    clearTimeout(timeout);
    clickedHourWeatherData.value = clickedDayWeatherData.value[hIndex];
    hourIcon.value = clickedHourWeatherData.value.icon;
    changeBackgroundClr(hIndex);
    timeout = setTimeout(() => {
        clickedDayWeatherData.value = forecastWeatherData.value[0];
        clickedHourWeatherData.value = clickedDayWeatherData.value[parseInt(locationTime.value.slice(12, 14), 10)];
        hourIcon.value = clickedHourWeatherData.value.icon;
        changeBackgroundClr(parseInt(locationTime.value.slice(12, 14), 10));
    }, 10000);
};

const getLocationsWeatherData = async () => {
    const weather_data = await fetch(weatherUrl.value).then((res) => res.json());
    const city_data = await fetch(cityUrl.value).then((res) => res.json());
    currentLocation.value =
        city_data.results[0].components.city ||
        city_data.results[0].components.state ||
        city_data.results[0].components.country ||
        "somewhere";
    timeZone.value = weather_data.timezone;

    locationTime.value = dateObj.value.toLocaleString("en-CA", {
        hour: "2-digit",
        minute: "2-digit",
        second: "2-digit",
        year: "numeric",
        month: "2-digit",
        day: "2-digit",
        hour12: false,
        timeZone: timeZone.value,
    });

    weatherUnits.value = {
        temp_unit: weather_data.hourly_units.temperature_2m,
        precipitation_unit: weather_data.hourly_units.precipitation,
        humidity_unit: weather_data.hourly_units.relativehumidity_2m,
        wind_unit: weather_data.hourly_units.windspeed_10m,
    };

    forecastWeatherData.value = {
        0: [],
        1: [],
        2: [],
        3: [],
        4: [],
        5: [],
        6: [],
    };
    for (let i = 0; i < weather_data.hourly.time.length; i++) {
        const index = parseInt(weather_data.hourly.time[i].slice(8, 10)) - parseInt(locationTime.value.slice(8, 10));
        forecastWeatherData.value[index].push({
            time: weather_data.hourly.time[i],
            temp: weather_data.hourly.temperature_2m[i],
            precipitation: weather_data.hourly.precipitation[i],
            humidity: weather_data.hourly.relativehumidity_2m[i],
            wind: weather_data.hourly.windspeed_10m[i],
            icon: setWeatherIcon(
                parseInt(weather_data.hourly.time[i].slice(11, 13)),
                parseFloat(weather_data.hourly.temperature_2m[i]),
                parseFloat(weather_data.hourly.precipitation[i]),
            ),
        });
    }
    handleDayWeatherData(0);
    handleHourWeatherData(parseInt(locationTime.value.slice(12, 14), 10));
};

onBeforeMount(() => {
    navigator.geolocation.getCurrentPosition(
        async (position) => {
            console.log(position.coords.latitude, position.coords.longitude);
            latitude.value = position.coords.latitude;
            longitude.value = position.coords.longitude;
            await getLocationsWeatherData();
        },
        (error) => console.log(error),
        {
            timeout: 1000,
        },
    );
});
onBeforeUnmount(() => {
    clearInterval(interval);
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
                        width: 150px;
                        height: 150px;
                        object-fit: contain;
                        margin: 40px 0;
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
                    .location-info__precipitation,
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
                                    flex: 0 0 calc(25% - 27.5px);
                                    text-align: center;
                                    font-size: 15.5px;
                                    padding: 10px;
                                    &:hover {
                                        background: rgba(37, 102, 163, 0.2);
                                        border-radius: 20px;
                                    }
                                    figure {
                                        height: 100px;
                                        display: flex;
                                        justify-content: center;
                                        align-items: center;
                                        img {
                                            width: 40px;
                                            height: auto;
                                        }
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
                            width: 100%;
                            section {
                                &:hover {
                                    background: rgba(37, 102, 163, 0.2);
                                }
                                .nextday {
                                    display: flex;
                                    align-items: center;
                                    justify-content: space-between;
                                    width: calc(100% - 36px);
                                    padding: 15px 0;
                                    margin: 5px auto;
                                    p {
                                        flex: 1;
                                    }
                                    img {
                                        width: 30px;
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
