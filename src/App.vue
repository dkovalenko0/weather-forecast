<template>
  <div id="app">
    <div class="weather-forecast">
      <div class="container">
        <h1 class="weather-logo">
          Weather forecast
        </h1>
        <div class="weather-search">
          <input type="text" placeholder="Enter city" v-model="city" />
          <button @click="getOneDayForecast">get one day forecast</button>
          <button @click="getFifeDayForecast">get fife day forecast</button>
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
                  {{ forecast.currentTemp | fixTemperature }}°
                </div>
                <div class="overcast">{{ forecast.overcast }}</div>
              </div>
            </div>
            <div>
              <span class="temp-values"
                >Min {{ forecast.minTemp | fixTemperature }}° <br />
                Max {{ forecast.maxTemp | fixTemperature }}°</span
              >
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
        <!-- <div v-else class="center"></div> -->
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

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
      }
    };
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

      this.forecast.city = "";
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
      this.loading = false;
    },
    getFifeDayForecast() {
      axios
        .get(
          "http://api.openweathermap.org/data/2.5/forecast?q=Kyiv&?units=metric&APPID=96879137844d6188d912d1665cb66f95"
        )
        .then(response => {
          console.log(response.data);
        });
    }
  },
  // mounted() {
  //   this.getOneDayForecast();
  // },
  filters: {
    fixTemperature: function(value) {
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
.weather-forecast {
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: peachpuff;

  .container {
    width: 500px;
  }

  .weather-logo {
    text-align: center;
  }

  .weather-search {
    display: flex;
    justify-content: space-evenly;

    input {
    }

    button {
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
