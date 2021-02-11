<template>
  <div id="app" >
    <main>
      <div v-if="settings" class="settings">
        <div class="setting-header">
          <p class="title">Setting</p>
          <button v-on:click="settings = !settings" class="gear">
            <img v-if="settings" src="https://cdn4.iconfinder.com/data/icons/ionicons/512/icon-close-512.png" alt="Close">
          </button>
        </div>
        <div class="setting-item-wrap">
          <draggable v-model="cities" ghost-class="ghost" @end="onEnd">
            <transition-group tag="div" type="transition" name="flip-list">
              <div :id="element.id" :key="element.id" v-for="element in cities" class="setting-item">
                <button class="bars">
                  <img src="https://cdn.iconscout.com/icon/free/png-512/bars-collection-view-application-grid-menu-44415.png" alt="Bars">
                </button>
                <div class="set-location">
                  {{ element.name }}
                </div>
                <button class="delete" v-on:click="deleteLocation(element)">
                  <img src="http://simpleicon.com/wp-content/uploads/trash.png" alt="Delete">
                </button>
              </div>
            </transition-group>
          </draggable>
        </div>
        <div class="add-location">
          <p>Add Location:</p>
          <div class="search-bar">
            <input
                type="text"
                class="search-bar"
                placeholder="Search..."
                v-model="query"
                @keypress="AddLocationEnter"
            >
            <button v-on:click="AddLocation" class="modal-btn">
              &crarr;
            </button>
          </div>
          <small v-if="!goodAnswer">
            This city does not exist
          </small>
        </div>
      </div>
      <div v-else >
        <div v-for="el in citiesMain" class="weather-box" :id="el.id" :key="el.id">
          <div class="weather-header">
            <div class="title">
              {{ el.name }}, {{ el.sys.country }}
            </div>
            <button v-on:click="settings = !settings" class="gear">
              <img src="https://freepngimg.com/thumb/gears/35357-7-gears-file-thumb.png" alt="gear">
            </button>
          </div>
          <div class="temp">
            <img v-bind:src="'http://openweathermap.org/img/w/' + el.weather[0].icon + '.png'" alt="Icon Weather">
            {{ Math.round(el.main.temp) }}°C</div>
          <div class="temp-feel">
            <strong>Feels like:</strong> {{ Math.round(el.main.feels_like) }}°C.
          </div>
          <div class="weather"><strong>{{ el.weather[0].main }}.</strong> ({{ el.weather[0].description }})</div>
          <div class="other-weather">
            <div class="wind">
              <img src="https://uxwing.com/wp-content/themes/uxwing/download/27-weather/wind.png"
                   alt="Wind">
              {{ el.wind.speed }}m/s</div>
            <div class="pressure">
              <img src="http://cdn.onlinewebfonts.com/svg/img_262976.png"
                   alt="Pressure">
              {{ el.main.pressure }}hPa </div>
            <div class="humidity"><strong>Humidity:</strong> {{ el.main.humidity }}% </div>
            <div class="visibility"><strong>Visibility:</strong> {{ Number(el.visibility / 1000).toFixed(1) }} km</div>
          </div>
        </div>
      </div>
    </main>
    <transition v-if="showModal" name="modal">
      <div class="modal-mask">
        <div class="modal-wrapper">
          <div class="modal-container">
            <div class="modal-body">
              <slot name="body">
                <p>Enter your city:</p>
                <div class="search-bar">
                  <input
                      type="text"
                      class="search-bar"
                      placeholder="Search..."
                      v-model="query"
                      @keypress="fetchWeatherEnter"
                  >
                  <button v-on:click="fetchWeather" class="modal-btn">
                    &crarr;
                  </button>
                </div>
              </slot>
            </div>
          </div>
        </div>
      </div>
    </transition>
    <!--    to ease dev-->
    <!--    <div style="position:fixed; top: 50px;left: 50px">-->
    <!--      <button style="padding: 10px;width: 150px;font-size: 18px;" id="show-modal"-->
    <!--              @click="showModal = !showModal">Show Modal</button>-->
    <!--      <button style="background:red;padding: 10px;width: 150px;font-size: 18px;"-->
    <!--              v-on:click="killCookies">-->
    <!--        KILL COOKIES-->
    <!--      </button>-->
    <!--    </div>-->
  </div>
</template>

<script>
import Vue from 'vue';
import Cookies from 'js-cookie';
import draggable from 'vuedraggable';

export default {
  name: 'App',
  data: function () {
    return {
      api_key: '30882e35a7090b8084166646c0eb45fc',
      url_base: 'http://api.openweathermap.org/data/2.5/',
      query: '',
      weather: {},
      weatherMain: {},
      showModal: true,
      settings: false,
      oldIndex: '',
      newIndex: '',
      cities: [],
      citiesMain: [],
      citiesCount: 0,
      cleanCookie: true,
      goodAnswer: true,
      newWeather: '' || undefined,
    }
  },
  components: {
    draggable,
  },
  mounted: function () {
    if (typeof Cookies.get('City') === 'undefined') {
      this.showModal = true;
      this.cleanCookie = true;
    } else {
      this.showModal = false;
      this.cleanCookie = false;
    }
    if (this.cleanCookie) {
    } else {
      this.cities = JSON.parse(Cookies.get('City'));
      this.citiesCount = JSON.parse(Cookies.get('CityID'))
      for (let i = 0; i < this.citiesCount; i++) {
        fetch(`${this.url_base}weather?q=${this.cities[i].name}&units=metric&APPID=${this.api_key}`)
            .then(res => {
              return res.json();
            }).then(this.setResultsCookie);
      }
    }
  },
  methods: {
    fetchWeather() {
      fetch(`${this.url_base}weather?q=${this.query}&units=metric&APPID=${this.api_key}`)
          .then()
          .then(res => {
            return res.json();
          }).then(this.setResults);
      this.showModal = false;
      this.query = '';
    },
    fetchWeatherEnter(e) {
      if (e.key == 'Enter') {
        this.fetchWeather()
      }
    },
    AddLocation() {
      fetch(`${this.url_base}weather?q=${this.query}&units=metric&APPID=${this.api_key}`)
          .then(res => {
            return res.json();
          }).then(this.setResults);
      this.query = '';
    },
    AddLocationEnter(e) {
      if (e.key == 'Enter') {
        this.AddLocation()
      }
    },
    setResultsCookie: function (results) {
      this.weather = results;
      this.citiesMain.push(this.weather);
    },
    setResults: function (results) {
      this.weather = results;
      if (typeof this.weather.name !== 'undefined') {
        this.goodAnswer = true;
        this.citiesMain.push(this.weather);
        let cityWeather = {
          name: this.weather.name,
          id: 0
        };
        if (typeof Cookies.get('CityID') === 'undefined') {
          cityWeather.id++;
          this.cities.push(cityWeather);
          Cookies.set('CityID', JSON.stringify(cityWeather.id))
        } else {
          cityWeather.id = +JSON.parse(Cookies.get('CityID'));
          cityWeather.id++;
          this.cities.push(cityWeather);
          Cookies.set('CityID', JSON.stringify(cityWeather.id))
        }
        Cookies.set('City', JSON.stringify(this.cities))
      } else {
        this.goodAnswer = false;
      }
    },
    deleteLocation(element) {
      this.cities = this.cities.filter(t => t.name !== element.name);
      this.citiesMain = this.citiesMain.filter(t => t.name !== element.name);
      Cookies.set('City', JSON.stringify(this.cities))
    },
    onEnd(event) {
      this.oldIndex = event.oldIndex;
      this.newIndex = event.newIndex;
      Cookies.remove('City');
      Cookies.set('City', JSON.stringify(this.cities));
    },
    // killCookies() {
    //   Cookies.remove('City');
    //   Cookies.remove('CityID');
    // },
  },
}
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Open+Sans&display=swap');
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
body {
  font-family: 'Open Sans', sans-serif;
}
.flip-list-move {
  transition: transform .5s;
}
main {
  width: 300px;
  margin: 0 auto;
  .weather-box, .settings {
    background: #fff;
    border: 1px solid #ccc;
    width: 300px;
    min-height: 350px;
    padding: 25px 25px 50px;
    .setting-header, .weather-header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-bottom: 20px;
      .title {
        color: #000;
        font-size: 20px;
        font-weight: 400;
      }
      .gear {
        border: none;
        outline: none;
        background: #fff;
        cursor: pointer;
        img {
          width: 25px;
        }
      }
    }
  }
  .settings {
    display: flex;
    flex-direction: column;
    .sortable-drag {
      opacity: 0;
    }
    .ghost {
      border: 3px solid #676767;
    }
    .setting-item-wrap{
      margin-bottom: 10px;
      .setting-item {
        display: flex;
        align-items: center;
        background: #ececec;
        padding: 9px;
        margin-bottom: 10px;
        .set-location {
          font-size: 17px;
          margin-left: 10px;
          line-height: 25px;
        }
        .delete {
          margin-left: auto;
        }
        .bars, .delete {
          border: none;
          outline: none;
          background: transparent;
          cursor: pointer;
          img {
            width: 20px;
          }
        }
        .bars {
          cursor: move;
        }
      }
    }
    .add-location {
      margin-top: auto;
      p {
        font-size: 20px;
        margin-top: 10px;
      }
      .search-bar {
        display: flex;
        align-items: center;
        input {
          padding: 8px;
          font-size: 17px;
          border: 2px solid #000;
          display: block;
        }
        input:focus {
          border-color: #8e86ff;
          outline: none;
        }
        button {
          outline: none;
          border: none;
          background: transparent;
          font-size: 30px;
          margin-left: 10px;
          line-height: 30px;
          display: block;
          width: 20px;
        }
      }
    }
  }
  .weather-box {
    margin-bottom: 30px;
    .temp {
      display: flex;
      align-items: center;
      justify-content: space-evenly;
      margin: 30px 0;
      font-size: 35px;
      font-weight: 700;
      img {
        width: 60px;
      }
    }
    .temp-feel, .weather {
      font-size: 15px;
      font-weight: normal;
    }
    .other-weather {
      display: flex;
      flex-wrap: wrap;
      justify-content: space-between;
      div {
        width: 50%;
        margin-top: 20px;
        text-align: left;
        font-size: 18px;
      }
      .wind, .pressure {
        display: flex;
        align-items: center;
        img {
          background: #fff;
          width: 25px;
          margin-right: 7px;
        }
      }
      .humidity, .visibility {
        font-size: 14px;
      }
    }
  }
}
.modal-mask {
  position: fixed;
  z-index: 9998;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: table;
  transition: opacity 0.3s ease;
  .modal-wrapper {
    display: table-cell;
    vertical-align: middle;
    .modal-container {
      width: 300px;
      margin: 0px auto;
      padding: 20px 30px;
      background-color: #fff;
      border-radius: 2px;
      box-shadow: 0 2px 8px rgba(0, 0, 0, 0.33);
      transition: all 0.3s ease;
      .modal-body {
        margin: 20px 0;
        p {
          font-size: 20px;
          margin-bottom: 15px;
        }
        .search-bar {
          display: flex;
          align-items: center;
          input {
            padding: 8px;
            font-size: 17px;
            border: 2px solid #000;
            display: block;
          }
          input:focus {
            border-color: #8e86ff;
            outline: none;
          }
          button {
            outline: none;
            border: none;
            background: transparent;
            font-size: 30px;
            margin-left: 10px;
            line-height: 30px;
            display: block;
            width: 20px;
          }
        }
      }
      .modal-default-button {
        float: right;
      }
      .modal-enter {
        opacity: 0;
      }
      .modal-leave-active {
        opacity: 0;
      }
      .modal-enter .modal-container,
      .modal-leave-active .modal-container {
        -webkit-transform: scale(1.1);
        transform: scale(1.1);
      }
    }
  }
}
</style>
