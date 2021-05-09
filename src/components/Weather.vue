<template>
  <div class="hello">
    {{ current_location_text }}
    <br />
    <br />
    Latitude: <input type="text" v-model="lat" /> Longitude:
    <input type="text" v-model="lon" />
    <br />
    <select v-model="current_option" @change="getWeather">
      <option v-for="option in options" :value="option" :key="option">
        {{ option }}
      </option>
    </select>
    <button @click="getWeather">Search</button>
    <br />
    <br />
    <div v-if="weathers.length">
      Timezone: {{ timezone }}
      <br />
      <hr />
      <div v-for="(weather, key) in weathers" :key="key">
        <div>
          {{
            dayjs(weather.dt * 1000).format(
              current_option === "hourly" ? "LLL" : "LL"
            )
          }}
        </div>
        <div v-for="wData in weather.weather" :key="wData.id">
          <div>{{ wData.main }} ({{ wData.description }})</div>
          <img :src="`http://openweathermap.org/img/wn/${wData.icon}.png`" />
        </div>
        <div v-if="weather.temp.min && weather.temp.max">
          min: {{ weather.temp.min }} ° C max: {{ weather.temp.max }} ° C
        </div>
        <div v-else-if="weather.temp">{{ weather.temp }} ° C</div>
        <hr />
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import dayjs from "dayjs";
var localizedFormat = require("dayjs/plugin/localizedFormat");
dayjs.extend(localizedFormat);
export default {
  name: "Weather",
  data() {
    return {
      dayjs,
      openweathermap_key: process.env.VUE_APP_OPENWEATHERMAP_KEY,
      lat: localStorage.getItem("lat"),
      lon: localStorage.getItem("lon"),
      current_option: "current",
      options: ["current", "hourly", "daily"],
      weathers: [],
      current_location_text: "",
      timezone: "",
    };
  },
  mounted() {
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(
        (position) => {
          this.current_location_text = `Your current Latitude: ${position.coords.latitude} and Longitude: ${position.coords.longitude}`;
          if (!this.lat && !this.lon) {
            this.lat = position.coords.latitude;
            this.lon = position.coords.longitude;
            this.getWeather();
          }
        },
        (error) => {
          switch (error.code) {
            case error.PERMISSION_DENIED:
              this.current_location_text =
                "User denied the request for Geolocation.";
              break;
            case error.POSITION_UNAVAILABLE:
              this.current_location_text =
                "Location information is unavailable.";
              break;
            case error.TIMEOUT:
              this.current_location_text =
                "The request to get user location timed out.";
              break;
            case error.UNKNOWN_ERROR:
              this.current_location_text = "An unknown error occurred.";
              break;
          }
        }
      );
    } else {
      this.current_location_text =
        "Geolocation is not supported by this browser.";
    }
    if (this.lat && this.lon) {
      this.getWeather();
    }
  },
  watch: {
    lat: function (value) {
      localStorage.setItem("lat", value);
    },
    lon: function (value) {
      localStorage.setItem("lon", value);
    },
  },
  methods: {
    getWeather() {
      this.weathers = [];
      axios
        .get("https://api.openweathermap.org/data/2.5/onecall", {
          params: {
            units: "metric",
            lat: this.lat,
            lon: this.lon,
            exclude: this.options
              .filter((op) => op !== this.current_option)
              .join(","),
            appid: this.openweathermap_key,
          },
        })
        .then((response) => {
          this.timezone = response.data.timezone;
          switch (this.current_option) {
            case "current":
              this.weathers = [response.data.current];
              break;
            case "hourly":
              this.weathers = response.data.hourly;
              break;
            case "daily":
              this.weathers = response.data.daily;
              break;
            default:
              break;
          }
        })
        .catch((err) => {
          alert(err.response.data.message);
        });
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
