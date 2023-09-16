<template>
  <div>
    <div class="container">
      <div class="row">
        <div class="col">
          <div class="input-group mb-3">
            <input v-model="cityName" type="text" class="form-control" @keydown.enter="getWeatherByName"
                   placeholder="Sisesta linna nimi"
                   aria-label="Username"
                   aria-describedby="basic-addon1">
          </div>

          <button type="button" class="btn btn-info" @click="getWeatherByName">Mis ilm on?</button>
          <button type="button" class="btn btn-info" @click="">Hakka statistikat koguma</button>

          <table class="table table-striped" v-show="(weatherResponse.main.temp > 0)">
            <thead>
            <tr>
              <th scope="col">Linn</th>
              <th scope="col">Temperatuur</th>
              <th scope="col">Tuulekiirus</th>
              <th scope="col">Õhuniiskus</th>
            </tr>
            </thead>
            <tbody>
            <tr>
              <td>{{ coordinatesResponse.name }}</td>
              <td>{{ tempCelsiusRounded }}°C</td>
              <td>{{ weatherResponse.wind.speed }} m/s</td>
              <td>{{ weatherResponse.main.humidity }}%</td>
            </tr>
            </tbody>
          </table>
        </div>

        <div class="col">
          <div class="dropdown">
            <button class="btn btn-secondary dropdown-toggle" type="button" id="dropdownMenuButton1"
                    data-bs-toggle="dropdown" aria-expanded="false">
              Vali linn
            </button>
            <ul class="dropdown-menu" aria-labelledby="dropdownMenuButton1">
              <li><a class="dropdown-item" href="#">kõik linnad</a></li>
            </ul>
          </div>
          <button type="button" class="btn btn-info" @click="">Lõpeta statistika kogumine</button>

          <table class="table table-striped">
            <thead>
            <tr>
              <th scope="col">Linn</th>
              <th scope="col">Temperatuur</th>
              <th scope="col">Tuulekiirus</th>
              <th scope="col">Õhuniiskus</th>
              <th scope="col">Aeg</th>
            </tr>
            </thead>
            <tbody>
            <tr>
              <td></td>
              <td></td>
              <td></td>
              <td></td>
              <td></td>
            </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// TODO: esimesel klikil ei lähe uue linna koordinaadid teele, vastus tuleb vana linna koordinaatidega. alles
// TODO: teisel klikil lähevad uue linna nimega

export default {
  name: 'HomeView',
  data() {
    return {
      apiKey: '160cd4ad5dbf4da0d772ab9a8bb1fa00',
      queryTime: 0,
      cityName: '',
      tempCelsiusRounded: 0,
      weatherResponse: {
        main: {
          temp: 0,
          humidity: 0,
        },
        wind: {
          speed: 0,
        },
        dt: 0,
        timezone: 0,
      },
      coordinatesResponse: {
        name: '',
        lat: 99,
        lon: 99,
      }
    }
  },
  methods: {
    async getWeatherByName() {
      await this.getCoordinatesByLocationName()
      await this.getWeatherDataByCoordinates()
      this.formatTime()
      this.formatTemperature()
    },
    getCoordinatesByLocationName() {
      const url = 'http://api.openweathermap.org/geo/1.0/direct'
      const citiesLimit = 1
      this.$http.get(url, {
            params: {
              q: this.cityName,
              limit: citiesLimit,
              appid: this.apiKey
            }
          }
      ).then(response => {
        this.coordinatesResponse = response.data[0]
      }).catch(error => {
        const errorResponseBody = error.response.data
      })
    },
    getWeatherDataByCoordinates() {
      const url = 'http://api.openweathermap.org/data/2.5/weather'
      this.$http.get(url, {
            params: {
              lat: this.coordinatesResponse.lat,
              lon: this.coordinatesResponse.lon,
              appid: this.apiKey
            }
          }
      ).then(response => {
        this.weatherResponse = response.data
      }).catch(error => {
        const errorResponseBody = error.response.data
      })
    },
    formatTime() {
      let unixTimestamp = this.weatherResponse.dt
      let date = new Date(unixTimestamp * 1000)
      this.queryTime = date.toLocaleTimeString()
    },
    formatTemperature() {
      this.tempCelsiusRounded = (this.weatherResponse.main.temp - 273.15).toFixed(1)
    },
  }
}
</script>