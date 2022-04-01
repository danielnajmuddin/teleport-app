<template>
  <div>
    Weather App
    <div
      v-if="weather.id"
    >
    <v-col
        class="d-flex"
        cols="12"
        sm="6"
      >
        <v-select
          :items="items"
          label="Seleact a City"
          v-model="selectCity"
          dense
          @change="cityWeather"
        ></v-select>
      </v-col>
    
    <p class="mb-0">
      {{weather.city}}, {{weather.country}}
      <v-icon
        dark
        right
      >
        mdi-map-marker
      </v-icon>
    </p>
    <h1 class="font-regular">{{(weather.main.feels_like).toFixed(0)}}<sup>&deg;</sup></h1>
    <p>{{weather.weather.description}}</p>
    <img :src=w_icon alt="weather icon">
  </div>
  </div>
</template>

<script>
export default {
  name: 'IndexPage',
  data() {
    return {
      items: ['Kuala Lumpur', 'George Town', 'Johor Bahru', 'Shah Alam', 'Putrajaya',
      'Cheras', 'Kajang', 'Klang', 'Rawang', 'Petaling Jaya', 'Semenyih', 'Banting',
      'Alor Setar', 'Sungai Petani', 'Kuching', 'Miri', 'Kota Kinabalu', 'Ipoh', 'Kuala Kangsar',
      'Kuala Terengganu', 'Dungun', 'Kota Bahru', 'Butterworth', 'Juru', 'Perai', 'Kuantan',
      'Seremban', 'Port Dickson', 'Alor Gajah', 'Ayer Keroh'],
      city: null,
      selectCity: "Kuala Lumpur",
      forecast: [],
      addCity: null,
      loading: false,
      current: {
        id: null,
      },
      API_KEY: "651072fb549d123c9df96253099daf2e",
    };
  },

  created() {
    this.cityWeather()
  },

  methods: {
    currentLocation() {
      if (navigator.geolocation) {
        this.loading = true;
        this.forecast = [];
        navigator.geolocation.getCurrentPosition(
          (position) => {
            this.coordsWeather(
              position.coords.latitude,
              position.coords.longitude
            );
            this.loading = false;
          },
        )
      }
    },
    cityWeather() {
      this.loading = true;
      this.forecastWeather();
      this.$axios
      .$get(
        `/data/2.5/weather?q=${this.selectCity}&units=metric&appid=${this.API_KEY}`
      )
      .then((response) => {
        this.current = {
          id: response.id,
          city: response.name,
          country: response.sys.country,
          weather: response.weather[0],
          main: response.main,
          wind: response.wind,
        };
        this.loading = false; 
        console.log(this.current)
      })
    },
    forecastWeather() {
      this.$axios
      .$get(
        `/data/2.5/forecast?q=${this.selectCity}&units=metric&appid=${this.API_KEY}`
      )
      .then((response) => {
        let list = response.list;
        this.forecast = [];
        for (let i = 0; i < list.length; i++) {
          let newDate = moment(list[i].dt_txt).format("D/MM");
          let now = moment(list[i].dt_txt).format("DDMMYYYYkkmmss");
          if (now > this.rightNow) {
            if (this.tempDate !== newDate) {
              this.tempDate = newDate;
              let p = {
                date: newDate,
                dateString: list[i].dt_txt,
                list: [],
              };
              this.forecast.push(p);
            }
            let obj = {
              date: list[i].dt,
              dateString: list[i].dt_txt,
              weather: list[i].weather[0].main,
            };
            let index = this.forecast.findIndex(
              (f) => f.date === this.tempDate
            );
            this.forecast[index].list.push(obj);
          }
        }
        this.tempDate = "";
      })
    
    },
    coordsWeather(lat, long) {
      console.log(lat, long)
      this.$axios
      .$get(
        `/data/2.5/weather?lat=${lat}&lon=${long}&units=metric&appid=${this.API_KEY}`
      )
      .then((response) => {
        this.current = {
          id: response.id,
          city: response.name,
          country: response.sys.country,
          weather: response.weather[0],
          main: response.main,
          wind: response.wind,
        };
      })
      .catch((error) => {
        this.loading = false;
        this.error = true;
        this.errorMsg = error.response.data.message;
      });
    },
    kelvinToCelcius(k) {
      return (k - 273.15).toFixed(0);
    },
  },

  computed: {
    weather() {
      return this.current;
    },

    w_icon() {
      return this.weather.weather
      ? `http://openweathermap.org/img/wn/${this.weather.weather.icon}@2x.png`
      : ''
    },
  },


}
</script>
