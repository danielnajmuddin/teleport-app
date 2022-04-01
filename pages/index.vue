<template>
  <div>
    Weather App
  </div>
</template>

<script>
export default {
  name: 'IndexPage',
  data() {
    return {
      city: null,
      selectCity: "Kuala Lumpur",
      forecast: [],
      addCity: null,
      loading: false,
      API_KEY: "651072fb549d123c9df96253099daf2e",
    };
  },

  created() {
    console.log("hello")
    this.currentLocation()
  },

  methods: {
    currentLocation() {
      // if (navigator.geolocation) {
        this.loading = true;
        this.forecast = [];
        navigator.geolocation.getCurrentPosition(
          (position) => {
            console.log('kjjkm')
            this.coordsWeather(
              position.coords.latitude,
              position.coords.longitude
            );
            this.loading = false;
            console.log(position)
          },
          (err) => {
            this.loading = false;
            this.error = true;
            this.errorMsg =
              err.code == 1
                ? "You have denied location permission"
                : err.message;
              console.log('kjjkm',err)
          },
        )
      // }
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
      })
      .catch((error) => {
        this.loading = false;
        this.error = true;
        this.errorMsg = error.response.data.message;
      });
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
      .catch((err) => {
        this.loading = false;
        this.error = true;
        this.errorMsg = err.response.data.message;
      });
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
  },


}
</script>
