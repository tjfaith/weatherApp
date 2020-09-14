<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">

      <q-input  
        v-model="search" 
        @keyup.enter="getWeaterBySearch"
        placeholder="search"
        dark
        borderless
        >
     

         <template v-slot:prepend>
          <q-icon @click="getLocation" name="my_location" />
        </template>

         <template v-slot:append>
          <q-btn @click="getWeaterBySearch" round dense flat icon="search" />
        </template>
      </q-input>
    </div>

<template v-if="weatherData">
    <div class="col text-white text-center">
      <div class="text-h4 text-weight-light">
        {{weatherData.name}}
      </div>
      <div class="text-h6 text-weight-light">
        {{weatherData.weather[0].main}}
      </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
        <span>{{Math.round(weatherData.main.temp)}}</span>
        <span class="text-h4 relative-position degree">&deg;c</span>
      </div>
    </div>

    <div class="col text-center">
      <img :src="`http://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`" alt="">
    </div>
</template>

<template v-else>
  <div class="col column text-center text-white">
    <div class="col text-h2 text-weight-thin">
      LareMedia<br>weather
    </div>
     <q-btn class="col" flat @click="getLocation">
      <q-icon left size="3em" name="my_location" />
      <div>Find My Location</div>
    </q-btn>
  </div>
</template>

    <div class="col skyline">
    <div>
 <q-btn class="homeBtn" color="white" @click="weatherData = null" round dense flat icon="home" />
</div>
</div>

  </q-page>
</template>

<script>
export default {
  name: 'PageIndex',
  data(){
    return{
      search:'',
      weatherData:null,
      lat:null,
      lon:null,
      apiUrl:'https://api.openweathermap.org/data/2.5/weather',
      apiKey:'c5f42f77164bdcbb7b926f5bc965d3f3'
    }
  },
  computed:{
    bgClass(){
      if (this.weatherData){
        if (this.weatherData.weather[0].icon.endsWith('n')){
          return 'bg-night'
        }
        else{
          return 'bg-day'
        }
      }
    }
  },
  methods:{
    getLocation(){
     
      this.$q.loading.show()
      if(this.$q.platform.is.electron){
        this.$axios.get('https://freegeoip.app/json', {
      headers: {
        'Access-Control-Allow-Origin': '*',
        'Content-Type': 'application/json',
      },
      withCredentials: true,
      credentials: 'same-origin',
    }).then(resopnse =>{
          this.lat = response.data.latitude
          this.lon = response.data.longitude
        this.getWeatherByCoords()
        });
      }else{
      navigator.geolocation.getCurrentPosition(position=>{
        this.lat = position.coords.latitude;
        this.lon = position.coords.longitude
        this.getWeatherByCoords()
      })
      }
    },
    getWeatherByCoords(){
      this.$q.loading.show()
      this.$axios.get(`${this.apiUrl}?lat=${this.lat}&lon=${this.lon}&appid=${this.apiKey}&units=metric`).then(response => {
      this.weatherData = response.data
      this.$q.loading.hide()
      }).catch(()=>{
        alert("Location not found")
      this.$q.loading.hide()

      })
    },
      getWeaterBySearch(){
      this.$q.loading.show()
      this.$axios.get(`${this.apiUrl}?q=${this.search}&appid=${this.apiKey}&units=metric`).then(response => {
      this.weatherData = response.data
      this.$q.loading.hide()
      }).catch(()=>{
      alert("Location not found")
      this.$q.loading.hide()

      })
    }
  }
}
</script>

<style lang="scss">

.q-page{
  background:linear-gradient(to bottom, rgb(0, 0, 0), #f36c43)
}

.bg-night{
  background:linear-gradient(to bottom, #232526, #414345)

}
.bg-day{
  background:linear-gradient(to bottom, #00b4db, #0083b0)

}
.degree{
  top:-44px
}
.skyline{
  flex:0 0 100px;
  background: url(../assets/skyline.png);
  background-size: contain;
  background-position: center bottom;
  justify-content: center;
}
.skyline div{

  text-align:center
}
.homeBtn{
  border:2px solid white
  
}
</style>
