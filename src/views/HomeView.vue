<template>
  <div>
    <div class="container">
      <div class="row">
        <div class="col">
          <div class="input-group mb-3">
            <input v-model="cityName" type="text" class="form-control mt-5" @keydown.enter="getManualWeather(cityName)"
                   placeholder="Sisesta linna nimi"
                   aria-label="Username"
                   aria-describedby="basic-addon1">
          </div>

          <button type="button" class="btn btn-info" @click="getManualWeather(cityName)">Mis
            ilm on?
          </button>

          <table v-show="manualWeatherResponse.main.humidity > 0" class="table" aria-describedby="">
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
              <td>{{ manualWeatherResponse.name }}</td>
              <td>{{ manualWeatherResponse.main.temp }}°C</td>
              <td>{{ manualWeatherResponse.wind.speed }} m/s</td>
              <td>{{ manualWeatherResponse.main.humidity }}%</td>
            </tr>
            </tbody>
          </table>

          <button v-show="manualWeatherResponse.main.humidity > 0" type="button" class="btn btn-info"
                  @click="addNameToRecordingCities">Hakka selle linna andmeid
            salvestama
          </button>

          <div v-show="warning.length > 0" class="alert alert-danger" role="alert">
            {{ warning }}
          </div>
        </div>

        <div class="col">
          <select v-model="selectedCity" @change="getRecordedWeatherData" class="form-select mt-5"
                  aria-label="Default select example">
            <option value="">Kõik linnad</option>
            <option v-for="city in recordingCities">
              {{ city.city }}
            </option>
          </select>
          <button type="button" class="btn btn-info m-3" @click="removeNameFromRecordingCities(selectedCity)">Lõpeta
            valitud
            linna andmete salvestamine
          </button>
          <button type="button" class="btn btn-outline-danger" @click="deleteRecordedData">Kustuta salvestatud andmed
          </button>

          <table v-if="recordedDataExists" class="table table-striped" aria-describedby="">
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
            <tr v-for="statistic in statisticsResponse" :key="statisticsResponse.time">
              <td>{{ statistic.city }}</td>
              <td>{{ statistic.temp }}°C</td>
              <td>{{ statistic.wind }} m/s</td>
              <td>{{ statistic.humidity }}%</td>
              <td>{{ statistic.time }}</td>
            </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'HomeView',
  data() {
    return {
      cityName: '', //lahtrisse trükitud
      selectedCity: '', //rippmenüüst valitud
      recordingCities: [
        {
          id: 0,
          city: ''
        }
      ],
      warning: '',
      tempCelsiusRounded: 0,
      recordedDataExists: false,
      weatherResponse: {
        main: {
          temp: 0,
          humidity: 0
        },
        wind: {
          speed: 0,
        },
        name: '',
      },
      manualWeatherResponse: {
        main: {
          temp: 0,
          humidity: 0
        },
        wind: {
          speed: 0,
        },
        name: '',
      },
      statisticsResponse: [
        {
          id: 0,
          city: '',
          temp: 0,
          wind: 0,
          humidity: 0,
          time: ''
        }
      ]
    }
  },
  methods: {
    getManualWeather(cityName) { //manuaalne
      this.resetWarning()
      this.$http.get("/manual/get-weather", {
            params: {
              city: cityName
            }
          }
      ).then(response => {
        this.manualWeatherResponse = response.data
      }).catch(error => {
        this.warning = 'Palun täpsusta otsingusõna!'
        setTimeout(() => {this.warning = ''}, 5000)
      })
    },
    getAutomaticWeather(cityName) { //automeeritud
      return this.$http.get('/manual/get-weather', {
        params: {
          city: cityName
        }
          }
      ).then(response => {
        this.weatherResponse = response.data;
      }).catch(error => {
        this.warning = 'Automaatse ilmapäringuga tekkis probleem!'
        setTimeout(() => {this.warning = ''}, 5000)
      })
    },
    resetWarning() {
      this.warning = ''
    },
    addNameToRecordingCities() {
      if (this.manualWeatherResponse.name.length !== 0) {
        this.saveCityName()
        alert('Alustame selle linna ilmaandmete kogumist!')
        this.resetThisCityName()
      } else {
        alert('Sisesta linna nimi!')
      }
      this.getRecordingCities()
    },
    saveCityName() {
      this.$http.post("/add", null, {
        params: {
          cityName: this.manualWeatherResponse.name
        }
      })
    },
    resetThisCityName() {
      this.cityName = ''
    },
    getRecordingCities() {
      this.$http.get("/get-cities")
          .then(response => {
            this.recordingCities = response.data
          })
    },
    removeNameFromRecordingCities() {
      if (this.selectedCity.length === 0) {
        alert('Vali linn!')
        return
      }
      this.deleteCityName();
      alert('Selle linna ilmaandmeid enam ei koguta!')
      this.resetSelectedCity()
      this.getRecordingCities()
      this.getRecordedWeatherData()
    },
    deleteCityName() {
      this.$http.delete("/remove", {
            params: {
              cityName: this.selectedCity
            }
          }
      )
    },
    async deleteRecordedData() {
      await this.$http.delete("/delete")
      this.recordedDataExists = false
      this.getRecordedWeatherData()
    },
    resetSelectedCity() {
      this.selectedCity = ''
    },
    getAndSaveRegularWeatherByNames() { //REGULAARNE TEGEVUS
      setInterval(async () => {
        for (let cityObj of this.recordingCities) {
          let cityName = cityObj.city
          await this.getAutomaticWeather(cityName)
          await this.recordWeatherData(cityName)
          this.getRecordedWeatherData()
        }
      }, 900000);
    },
    recordWeatherData(cityName) {
      return this.$http.post("/record", null, {
            params: {
              city: cityName,
              temp: this.weatherResponse.main.temp,
              wind: this.weatherResponse.wind.speed,
              humidity: this.weatherResponse.main.humidity,
            }
          }
      )
    },
    getRecordedWeatherData() {
      this.$http.get("/show", {
            params: {
              city: this.selectedCity
            }
          }
      ).then(response => {
        if (response.data.length > 0) {
          this.recordedDataExists = true
          this.statisticsResponse = response.data
        }
      })
    },
  },
  mounted() {
    this.getRecordingCities()
    this.getAndSaveRegularWeatherByNames();
    this.getRecordedWeatherData()
  }
}
</script>