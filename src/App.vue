<!-- Use preprocessors via the lang attribute! e.g. <template lang="pug"> -->
<template>
  <div id="app">
    <section>
      <div class="container">
        <header>
          <div class="header__location">
            <img src="./components/icons/location.png" />
            <div>
              <p>Fortaleza</p>
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
            <p class="todays-weather__temp">20C</p>
            <p class="todays-weather__precipitations">Precipitations</p>
            <div class="todays-weather__minmax">
              <p id="max">Max.: 31º</p>
              <p id="min">Min.: 25º</p>
            </div>
          </div>
          <div class="location-info">
            <div class="location-info__rain">
              <img src="./components/icons/liquid.png" />
              <p>6%</p>
            </div>
            <div class="location-info__humidity">
              <img src="./components/icons/humidity.png" />
              <p>90%</p>
            </div>
            <div class="location-info__wind">
              <img src="./components/icons/wind.png" />
              <p>19km/h</p>
            </div>
          </div>
          <div class="today-forecast">
            <div class="today">
              <h1>Today</h1>
              <p class="todays__date">Mar, 9</p>
            </div>
            <ul class="forecast">
              <li>
                <p class="forecast__hour">29C</p>
                <img src="./components/icons/cloud-with-sun.png" />
                <p class="forecast__hour">15.00</p>
              </li>
              <li>
                <p class="forecast__hour">29C</p>
                <img src="./components/icons/cloud-with-sun.png" />
                <p class="forecast__hour">15.00</p>
              </li>
              <li>
                <p class="forecast__hour">29C</p>
                <img src="./components/icons/cloud-with-sun.png" />
                <p class="forecast__hour">15.00</p>
              </li>
              <li>
                <p class="forecast__hour">29C</p>
                <img src="./components/icons/cloud-with-sun.png" />
                <p class="forecast__hour">15.00</p>
              </li>
            </ul>
          </div>
        </main>
      </div>
    </section>
  </div>
</template>

<script setup>
import { ref, onBeforeMount } from "vue";
const data = ref(null);
const url = ref(
  "https://api.open-meteo.com/v1/forecast?latitude=10.82&longitude=106.63&hourly=temperature_2m"
);
const latitude = ref(null);
onBeforeMount(async () => {
  try {
    data.value = await (await fetch(url.value)).json();
    console.log(data.value);
    latitude.value = data.value.latitude;
  } catch (e) {
    console.log(e.message);
  }
});
</script>

<!-- Use preprocessors via the lang attribute! e.g. <style lang="scss"> -->
<style lang="scss">
#app {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100vw;
  height: 100vh;
  background-color: black;
  section {
    width: 423px;
    height: 858px;
    background: linear-gradient(
      167.44deg,
      #08244f 0%,
      #134cb5 47.38%,
      #0b42ab 100%
    );
    border-radius: 40px;
    .container {
      width: 80%;
      margin: 45px auto 0 auto;
      height: 50%;
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
        .today-forecast {
          font-size: 18px;
          margin: 20px 0;
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
            width: calc(100% - 36px);
            margin: 0 auto 12px auto;
            h1 {
              font-weight: 700;
              font-size: 20px;
            }
          }
          .forecast {
            display: flex;
            gap: 12px;
            justify-content: center;
            margin: 0 auto;
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
    }
  }
}
</style>
