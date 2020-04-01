<template>
  <div id="app">
    <div class="weather-forecast">
      <div class="container">
        <h1 class="weather-logo">Weather forecast</h1>
        <div class="weather-search">
          <input type="text" placeholder="Enter city" v-model="city" />
          <button @click="getOneDayForecast">1 DAY</button>
          <button @click="getFifeDayForecast">16 DAY</button>
        </div>
        <p v-if="error" class="center">{{ error }}</p>
        <div class="weather-item" v-else-if="forecast.cityName">
          <Loader v-if="loading" class="center" />
          <div class="weather" v-else>
            <div class="city">{{ forecast.cityName }}</div>
            <div class="weather-main-info">
              <img :src="forecast.icon" alt="icon" class="weather-icon" />
              <div>
                <div class="temperature">
                  {{ forecast.currentTemp | fixNumber }}°
                </div>
                <div class="overcast">{{ forecast.overcast }}</div>
              </div>
            </div>
            <div>
              <span class="temp-values">
                Min {{ forecast.minTemp | fixNumber }}°
                <br />
                Max {{ forecast.maxTemp | fixNumber }}°
              </span>
            </div>
          </div>
          <div class="info">
            <img class="icon" src="./assets/svg/021-sunrise.svg" />
            {{ forecast.sunrise }} AM
            <img class="icon" src="./assets/svg/021-sunset.svg" />
            {{ forecast.sunset }} PM
            <img class="icon" src="./assets/svg/humidity.svg" />
            {{ forecast.humidity }}
            <img class="icon" src="./assets/svg/wind.svg" />
            {{ forecast.wind }}
          </div>
        </div>
        <swiper
          class="swiper"
          :options="swiperOption"
          v-else-if="forecast16.length"
        >
          <swiper-slide
            v-for="f of forecast16"
            :key="f.wind_spd"
            class="weather-item"
          >
            <Loader v-if="loading" class="center" />
            <div class="weather">
              <div class="city">{{ f.cityName }}</div>
              <div class="weather-main-info">
                <img :src="f.icon" alt="icon" class="weather-icon" />
                <div>
                  <div class="temperature">
                    {{ f.currentTemp | fixNumber }}°
                  </div>
                  <div class="overcast">{{ f.overcast }}</div>
                </div>
              </div>
              <div>
                <span class="temp-values">
                  Min {{ f.minTemp | fixNumber }}°
                  <br />
                  Max {{ f.maxTemp | fixNumber }}°
                </span>
              </div>
            </div>
            <div class="info">
              <img class="icon" src="./assets/svg/021-sunrise.svg" />
              {{ f.sunrise }} AM
              <img class="icon" src="./assets/svg/021-sunset.svg" />
              {{ f.sunset }} PM
              <img class="icon" src="./assets/svg/humidity.svg" />
              {{ f.humidity }}
              <img class="icon" src="./assets/svg/wind.svg" />
              {{ f.wind }}
            </div>
          </swiper-slide>
          <div class="swiper-pagination" slot="pagination"></div>
        </swiper>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { Swiper, SwiperSlide } from "vue-awesome-swiper";
import "swiper/css/swiper.css";

export default {
  data() {
    return {
      city: "",
      error: "",
      loading: true,
      forecast: {
        cityName: "",
        currentTemp: "",
        minTemp: "",
        maxTemp: "",
        sunrise: "",
        sunset: "",
        humidity: "",
        wind: "",
        overcast: "",
        icon: ""
      },
      forecast16: [],
      swiperOption: {
        spaceBetween: 30,
        pagination: {
          el: ".swiper-pagination",
          type: "fraction"
        }
      }
    };
  },
  components: {
    Swiper,
    SwiperSlide
  },

  methods: {
    getOneDayForecast() {
      this.error = "";

      axios
        .get(
          `http://api.openweathermap.org/data/2.5/weather?q=${this.city}&units=metric&APPID=96879137844d6188d912d1665cb66f95`
        )
        .then(response => {
          this.forecast.city = "";
          this.forecast.cityName = response.data.name;
          this.forecast.currentTemp = response.data.main.temp;
          this.forecast.minTemp = response.data.main.temp_min;
          this.forecast.maxTemp = response.data.main.temp_max;
          this.forecast.humidity = response.data.main.humidity + " %";
          this.forecast.wind = response.data.wind.speed + " M/S";
          this.forecast.overcast = response.data.weather[0].description;
          this.forecast.icon = `http://openweathermap.org/img/wn/${response.data.weather[0].icon}@2x.png`;
          this.forecast.sunrise = new Date(response.data.sys.sunrise * 1000)
            .toLocaleTimeString("ru-RU")
            .slice(1, 5);
          this.forecast.sunset = new Date(response.data.sys.sunset * 1000)
            .toLocaleTimeString("ru-RU")
            .slice(1, 5);
        })
        .catch(e => {
          this.error = e.response.data.message;
        });

      this.forecast.cityName = "";
      this.forecast.currentTemp = "";
      this.forecast.minTemp = "";
      this.forecast.maxTemp = "";
      this.forecast.humidity = "";
      this.forecast.wind = "";
      this.forecast.overcast = "";
      this.forecast.icon = "";
      this.forecast.sunrise = "";
      this.forecast.sunset = "";
      this.forecast16 = [];
      this.loading = false;
    },
    getFifeDayForecast() {
      this.error = "";
      axios
        .get(
          `https://api.weatherbit.io/v2.0/forecast/daily?city=${this.city}&key=a56a8a3b0b9145e399a280a541f5aa5f`
        )
        .then(response => {
          for (let i = 0; i < response.data.data.length; i++) {
            this.forecast16.push({
              cityName: response.data.city_name,
              currentTemp: response.data.data[i].temp,
              minTemp: response.data.data[i].min_temp,
              maxTemp: response.data.data[i].max_temp,
              sunrise: new Date(response.data.data[i].sunrise_ts * 1000)
                .toLocaleTimeString("ru-RU")
                .slice(1, 5),
              sunset: new Date(response.data.data[i].sunset_ts * 1000)
                .toLocaleTimeString("ru-RU")
                .slice(1, 5),
              humidity: response.data.data[i].rh + " %",
              wind: response.data.data[i].wind_spd.toFixed(0) + " M/S",
              overcast: response.data.data[i].weather.description,
              icon: `https://www.weatherbit.io/static/img/icons/${response.data.data[i].weather.icon}.png`
            });
          }
        })
        .catch(e => {
          this.error = e.response.data.error;
        });

      this.forecast.cityName = "";
      this.forecast.currentTemp = "";
      this.forecast.minTemp = "";
      this.forecast.maxTemp = "";
      this.forecast.humidity = "";
      this.forecast.wind = "";
      this.forecast.overcast = "";
      this.forecast.icon = "";
      this.forecast.sunrise = "";
      this.forecast.sunset = "";
      this.forecast16 = [];
      this.loading = false;
    }
  },
  filters: {
    fixNumber: function(value) {
      return value.toFixed(0);
    }
  }
};
</script>

<style lang="scss">
body {
  font-family: "Montserrat", sans-serif;
  font-weight: 100;
  text-shadow: 0px 0px 2px #000000;
  color: #000;
  margin: 0;
}

.center {
  text-align: center;
}

.swiper {
  padding-bottom: 50px;
}
.weather-forecast {
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  background-color: peachpuff;

  .container {
    width: 500px;
  }

  .weather-logo {
    margin-top: 100px;
    margin-bottom: 50px;
    text-align: center;
  }

  .weather-search {
    display: flex;
    justify-content: space-evenly;

    input {
      font-family: "Montserrat", sans-serif;
      font-weight: 100;
      text-shadow: 0px 0px 2px #000000;
      color: #000;
      background-color: rgba(tan, 0.5);
      outline: none;
      border: none;
      border-radius: 15px;
      padding: 5px 10px;
    }

    button {
      font-family: "Montserrat", sans-serif;
      font-weight: 100;
      text-shadow: 0px 0px 2px #000000;
      color: #000;
      background-color: tan;
      outline: none;
      cursor: pointer;
      border: none;
      border-radius: 15px;
      padding: 5px 10px;
    }
  }

  .weather-item {
    margin-top: 50px;
    width: 500px;
    height: 200px;
    background-color: rgba(tan, 0.5);
    border-radius: 20px;

    display: flex;
    flex-direction: column;
    justify-content: space-around;
    align-items: center;

    .weather {
      width: inherit;
      display: flex;
      justify-content: space-around;
      align-items: center;

      .weather-main-info {
        display: flex;
        align-items: center;

        .weather-icon {
          margin-right: 10px;
        }

        .temperature {
          margin-bottom: 12px;
        }
      }
    }

    .info {
      width: inherit;
      display: flex;
      justify-content: space-evenly;
      align-items: center;

      .icon {
        width: 40px;
      }
    }
  }
}
</style>
