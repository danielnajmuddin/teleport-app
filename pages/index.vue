<template>
  <div>
    <div
      v-if="weather.id"
    >
    <div class="my-2">
          <v-btn
            color="primary"
            fab
            small
            dark
            @click="currentLocation"
          >
            <v-icon>mdi-crosshairs-gps</v-icon>
          </v-btn>
        </div>
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
    
    <img :src=w_icon alt="weather icon">
    <p>{{weather.weather.description}}</p>

      <div
      class="mt-10"
      v-if="forecast && forecast.length > 0"
    >
      <div
        v-for="(f, index) in forecast"
        :key="index"
      >
        <p class="font-semibold">{{formatDate(f.dateString)}}, {{formatDay(f.dateString)}}</p>
        <v-row
          class="justify-space-between"
          v-for="(l, i) in f.list"
          :key="i"
        >
          <v-col>
            <p>{{l.weather}}</p>
          </v-col>
          <v-col class="text-right">
            <p class="small-font">{{formatTime(l.dateString)}}</p>
          </v-col>
        </v-row>
      </div>
    </div>

    <v-carousel
    cycle
    height="400"
    hide-delimiter-background
    show-arrows-on-hover
  >
    <v-carousel-item
      v-for="(item, i) in items.slice(0, 3)"
      :key="i"
    >
        <v-row  
          class="fill-height"
          align="center"
          justify="center"
        >
          <div >
            {{ item }}
            <h3 class="font-regular">{{(weather.main.feels_like).toFixed(0)}}<sup>&deg;</sup></h3>
            <img :src=w_icon alt="weather icon">
            <p>{{weather.weather.description}}</p>
          </div>
        </v-row>
    </v-carousel-item>
  </v-carousel>

  </div>
  </div>
</template>

<script>
import moment from "moment"
export default {
  name: 'IndexPage',
  data() {
    return {
      items: ['Kuala Lumpur', 'George Town', 'Johor Bahru', 'Shah Alam', 'Putrajaya',
      'Cheras', 'Kajang', 'Klang', 'Rawang', 'Petaling Jaya', 'Semenyih', 'Banting',
      'Alor Setar', 'Sungai Petani', 'Kuching', 'Miri', 'Kota Kinabalu', 'Ipoh', 'Kuala Kangsar',
      'Kuala Terengganu', 'Dungun', 'Kota Bahru', 'Butterworth', 'Juru', 'Perai', 'Kuantan',
      'Seremban', 'Port Dickson', 'Alor Gajah', 'Ayer Keroh'],
      selectCity: "Kuala Lumpur",
      forecast: [],
      addCity: null,
      loading: false,
      tempDate: "",
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
    formatDay(d) {
      let dd = new Date(d);
      return moment(dd).format("dddd");
    },
    formatDate(d) {
      let dd = new Date(d);
      return moment(dd).format("D/MM");
    },
    formatTime(d) {
      let dd = new Date(d);
      return moment(dd).format("h:mma");
    },
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
            console.log(obj)
          }
        }
        this.tempDate = "";
      })
    
    },
    coordsWeather(lat, long) {
      console.log(lat, long)
      this.forecastWeather();
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
    convertTemperature(t) {
      return (t - 273.15).toFixed(0);
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

    rightNow() {
      return moment(new Date()).format("DDMMYYYYkkmmss");
    },
  },


}
</script>
