<script>
export default {
  name: 'app',
  data() {
    return {
      //POSITION
      options: {
        enableHighAccuracy: true,
        timeout: 5000,
        maximumAge: 0
      },
      //DATE
      months: ['Janvier', 'Février', 'Mars', 'Avril', 'Mai', 'Juin', 'Juillet', 'Août', 'Septembre', 'Octobre', 'Novembre', 'Décembre'],
      days: ['Dimanche','Lundi','Mardi','Mercredi','Jeudi','Vendredi','Samedi'],
      //API
      APIkey: "c1726c5d5ee8dac60d8c8a8cef8b8c57",
      cityURL: "http://api.openweathermap.org/geo/1.0/direct?q=",
      baseURL: "https://api.openweathermap.org/data/2.5/weather?",
      reverseURL: "http://api.openweathermap.org/geo/1.0/reverse?",
      searchValue: '',
      location: '',
      date: '',
      temperature: '',
      weather: '',
      cities: [],
      weatherResult: '',
      loading: false,
      timeout: null,
    }
  },
  computed: {
    getFullDate() {
      const today = new Date();
      console.log(today.getDay())
      return this.days[today.getDay()] + " "
            + today.getDate() + " "
            + this.months[today.getMonth()] + " "
            + today.getFullYear();
    },
  },
  mounted() {
    this.getCurrentPosition();
  },
  methods: {
    fetchCity() {
      if(this.searchValue.length < 3) {
        this.cities = [];
        return;
      }

      if(this.timeout)
      {
        clearTimeout(this.timeout);
        this.timeout = null;
      }

      fetch(this.cityURL
                  +this.searchValue
                  +"&limit=20"
                  + "&appid=" + this.APIkey)
          .then((response) => response.json())
          .then((data) => this.createCitiesList(data));

    },
    fetchWeather() {
      fetch(this.baseURL
          + "q=" + this.location.trim()
          + "&appid=" + this.APIkey
          + "&units=metric"
          + "&lang=fr")
          .then((response) => response.json())
          .then((data) => this.weatherResult = data);
    },
    fetchWeatherCoord(lat, lon) {
      fetch(this.reverseURL
          + "lat=" + lat
          + "&lon=" + lon
          + "&appid=" + this.APIkey)
          .then((response) => response.json())
          .then((data) => {
                let result = data[0];
                this.location = result.local_names.fr || result.name;
                fetch(this.baseURL
                    + "lat=" + lat
                    + "&lon=" + lon
                    + "&appid=" + this.APIkey
                    + "&units=metric"
                    + "&lang=fr")
                    .then((response) => response.json())
                    .then((data) => this.weatherResult = data)
              }
          );
    },
    submitCity(city) {
      this.searchValue = city || this.searchValue;
      this.cities = [];
      this.location = city;
      this.fetchWeather();
    },
    createCitiesList(data) {
      this.cities = [];
      data.forEach(elem =>
          {
            let display = "";
            if(elem.local_names)
              display = elem.local_names.fr;
            else
              display = elem.name;
            display += ", " + elem.country;
            if(elem.state)
              display += ", " + elem.state;
            this.cities.push(display);
          }
      )
    },
    capitalize(string) {
      return string.charAt(0).toUpperCase() + string.slice(1);
    },
    success(pos) {
      console.log(pos)
      const crd = pos.coords;
      this.fetchWeatherCoord(crd.latitude, crd.longitude);
      this.loading = false;
    },

    error(err) {
      console.warn(`ERROR(${err.code}): ${err.message}`);
      this.loading = false;
    },

    getCurrentPosition() {
      this.loading = true;
      this.timeout = setTimeout(() => this.loading = false, this.options.timeout);

      navigator.geolocation.getCurrentPosition(pos => this.success(pos),
          err => this.error(err),
          this.options);
    },
  },
  watch: {
    weatherResult(newValue) {
      console.log(this.weatherResult)
      this.temperature = Math.round(newValue.main.temp);
      this.weather = this.capitalize(newValue.weather[0].description);
    }
  }
}
</script>

<template>
  <div class="container">
    <div class="search-wrap">
      <form class="search-form" @submit.prevent="">
        <input  v-model="searchValue"
                class="search-bar"
                type="text"
                placeholder="Rechercher ..."
                @input="fetchCity"
                @focus="searchValue = ''">
      </form>
      <div class="table-wrap">
        <table class="cities-table">
          <tr v-for="(city,key) of cities" :key="key" @click="submitCity(city)">
            <td class="city">{{ city }}</td>
          </tr>
        </table>
      </div>
    </div>

    <div class="data-wrap">
      <span id="location">{{ location.split(`,`)[0] }}</span>
      <span id="date">{{ getFullDate }}</span>
      <div id="temperature-wrap">
        <span v-if="temperature">{{ temperature + "°C" }}</span>
        <span class="lds-dual-ring" v-else-if="loading"></span>
        <span id="refresh" v-else @click="getCurrentPosition">
          <img src="./assets/autorenew.svg">
        </span>
      </div>
      <span id="weather">{{ weather }}</span>
    </div>
  </div>
</template>

<style scoped>
.container {
  position: relative;
  background-image: url("./assets/cold-bg.jpg");
  background-size: cover;
  height: 100vh;
  padding: 25px;
}

.container::before {
  content: "";
  inset: 0;
  position: absolute;
  background-image: linear-gradient(to bottom, rgba(0,0,0,0.25), rgba(0,0,0,0.75));
  z-index: 1;
}

.search-wrap{
  position: relative;
  z-index: 5;
}

.search-bar {
  box-sizing:border-box;
  width: 100%;
  height: 50px;
  border: transparent;
  background-color: rgba(255, 255, 255, 0.66);
  border-radius: 0 15px 0 15px;
  font-size: 20px;
  color: #313131;
  padding: 15px;
  box-shadow: 0 0 8px rgba(0, 0, 0, 0.25);
  transition: 0.5s;
}

.search-bar:focus {
  border-radius: 15px 0 15px 0;
  box-shadow: 0 0 8px rgba(0, 0, 0, 0.25);
  outline: none;
}

.data-wrap {
  position: relative;
  margin-top: 20px;
  z-index: 2;
}

.data-wrap * {
  color: white;
}

#location {
  display: block;
  text-align: center;
  font-size: 40px;
  text-shadow: 2px 2px rgba(0, 0, 0, 0.25);
  font-weight: 600;
}

#date {
  display: block;
  text-align: center;
  font-size: 20px;
  font-style: italic;
  margin-top: 10px;
}

#temperature-wrap {
  text-align: center;
  margin-top: 20px;
}

#temperature-wrap span{
  display: inline-block;
  padding: 15px;
  border-radius: 15px;
  background-color: rgba(255, 255, 255, 0.25);
  font-size: 60px;
  box-shadow: 2px 4px rgba(0, 0, 0, 0.33);
  text-shadow: 2px 4px rgba(0, 0, 0, 0.25);
  font-weight: 900;
}

#refresh{
  font-size: 0px !important;
}

#refresh img{
  width: 60px;
  height: 60px;
}

#refresh:hover{
  background-color: rgba(255, 255, 255, 0.4);
}

#weather {
  display: block;
  margin-top: 15px;
  text-align: center;
  font-size: 30px;
  font-style: italic;
  font-weight: 750;
  text-shadow: 2px 3px rgba(0, 0, 0, 0.25);
}

.table-wrap {
  position: absolute;
  margin-top: 5px;
  /*max-height: 196px;*/
  overflow: auto;
  z-index: 10;
  width: 100%;
}

.cities-table {
  background-color: rgba(0, 0, 0, 0.8);
  color: white;
  width: 100%;
}

.cities-table td{
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  padding: 8px 15px 8px 15px;
}

.cities-table tr:first-child td{
  border-top: none;
}

.cities-table td:hover{
  background-color: rgba(255, 255, 255, 0.2);
}

.lds-dual-ring {
  display: inline-block;
  width: 80px;
  height: 80px;
  /*padding: 8px;*/
}
.lds-dual-ring:after {
  content: " ";
  display: block;
  width: 64px;
  height: 64px;
  border-radius: 50%;
  border: 6px solid #fff;
  border-color: #fff transparent #fff transparent;
  animation: lds-dual-ring 1.2s linear infinite;
}
@keyframes lds-dual-ring {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

</style>

<style>
body {
  margin: 0;
  font-family: "Montserrat Thin";
}
</style>
