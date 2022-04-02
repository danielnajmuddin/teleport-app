<template>
  <div class="bg">
    <div
      v-if="weather.id"
    >
    <div class="column justify-space-between">
      <v-select
        :items="items"
        label="Select a City"
        v-model="selectCity"
        dense
        @change="cityWeather"
      ></v-select>

      <v-text-field label="Add new city" v-model="newCity" append-icon="mdi-plus"
        @click:append="addCity">
      </v-text-field>

      <v-carousel
        cycle
        height="300"
        width="200"
        hide-delimiter-background
        show-arrows-on-hover
      >
        <v-carousel-item
          v-for="(m, i) in mainCities"
          :key="i"
        >
             <v-sheet
              :color="colors[i]"
              height="100%"
            >
            <v-row  
              class="fill-height"
              align="center"
              justify="center"
            >
              <div align="Center">
                <p>{{ m.city }}</p>
                <h1 class="font-regular">{{(m.main.feels_like).toFixed(0)}}<sup>&deg;</sup></h1>
                <p>{{ m.weather.main }}</p>
                <p>Humidity: {{m.main.humidity}} % | Wind: {{m.wind.speed}} ms</p>
              </div>
            </v-row>
            </v-sheet>
        </v-carousel-item>

      </v-carousel>
      <v-btn
        depressed
        color="primary"
        @click="currentLocation"
      >
        Show Current Location
      </v-btn>
    </div>
          
    <div align="center">
      <p >
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
      <p>Humidity: {{weather.main.humidity}} % | Wind: {{weather.wind.speed}} ms</p>
      <v-divider></v-divider>
    </div>
    
    

      <div
      class="mt-10"
      v-if="forecast && forecast.length > 0"
    >
      <div
        v-for="(f, index) in forecast"
        :key="index"
      >
        <p >{{formatDay(f.dateString)}}, {{formatDate(f.dateString)}}</p>
        <v-row
          class="justify-space-between"
          v-for="(l, i) in f.list"
          :key="i"
        >
          <v-col>
            <p>{{l.weather}}</p>
          </v-col>
          <v-col class="text-right">
            <p >{{formatTime(l.dateString)}}</p>
        
          </v-col>
        </v-row>
        <v-divider></v-divider>
      </div>
    </div>
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
      mainCities: [],
      forecast: [],
      newCity: null,
      loading: false,
      tempDate: "",
      current: {
        id: null,
      },
      currentTab: {
        id: null,
      },
      colors: [
          'yellow darken-4',
          'red lighten-1',
          'deep-purple accent-4'],
      API_KEY: "651072fb549d123c9df96253099daf2e",
    };
  },

  created() {
    this.cityWeather()
    this.callCarousel("Kuala Lumpur")
    this.callCarousel("George Town")
    this.callCarousel("Johor Bahru")
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
          if (now > this.currently) {
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
    },
    weatherByCity(mainCity) {
      this.$axios
      .$get(
        `/data/2.5/weather?q=${mainCity}&units=metric&appid=${this.API_KEY}`
      )
      .then((response) => {
        this.currentTab = {
          id: response.id,
          city: response.name,
          country: response.sys.country,
          weather: response.weather[0],
          main: response.main,
          wind: response.wind,
        };
        return response
      })
    },

    callCarousel(city) {
      this.$axios
      .$get(
        `/data/2.5/weather?q=${city}&units=metric&appid=${this.API_KEY}`
      )
      .then((response) => {
        let obj = {
          id: response.id,
          city: response.name,
          country: response.sys.country,
          weather: response.weather[0],
          main: response.main,
          wind: response.wind,
        };
        this.mainCities.push(obj)
      })
    },

    convertTemperature(t) {
      return (t - 273.15).toFixed(0);
    },

    addCity() {
      this.items.push(this.newCity)
      this.selectCity = this.newCity;
      this.cityWeather()
      this.newCity = null
    },
    
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

  },

  computed: {
    weather() {
      return this.current;
    },

    mainweather() {
      return this.currentTab;
    },

    w_icon() {
      return this.weather.weather
      ? `http://openweathermap.org/img/wn/${this.weather.weather.icon}@2x.png`
      : ''
    },

    currently() {
      return moment(new Date()).format("DDMMYYYYkkmmss");
    },
  },


}
</script>

<style lang="scss">
.bg {
  padding: 20px 10px;
  margin: 10px;
}
</style>