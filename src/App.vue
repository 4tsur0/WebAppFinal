<template>
  <div class="wrapper" v-cloak>
    <header>
      <p>Next Event (Demo)</p>
      <div class="status">
        <ul>
          <li>Status</li>
          <li>TheSportsDB : {{ apiStatus.thesportdb }}</li>
          <li>Mapbox : {{ apiStatus.mapbox }}</li>
          <li>OpenWeather : {{ apiStatus.openweather }}</li>
        </ul>
      </div>
    </header>

    <main>
      <div v-if="status === -0.5" class="info">
        <h2>Cannot get SPORTS data </h2>
        <h3>Your TheSportDB API Key in not VALID</h3>
        <h3>Please set VALID Key</h3>
      </div>

      <div>
        <multiselect
          v-if="status > 0"
          v-model="sport"
          :options="allSports"
          :show-labels="false"
          placeholder="Pick a sport"
          :allow-empty="false"
          @input="selectSport()"
        >
        </multiselect>

        <multiselect
          v-if="status > 1" :key="this.renderKey.leagues"
          v-model="league"
          :options="filtedLeagues"
          label="strLeague"
          :show-labels="false"
          placeholder="Pick a league"
          :allow-empty="false"
          noOptions="Sport is NOT found"
          @input="selectLeague()"
        >
        </multiselect>

        <multiselect
          v-if="status > 2 && selectedTeam !== null" :key="this.renderKey.teams"
          v-model="selectedTeam"
          placeholder="Pick a team"
          label="strTeam"
          :options="allTeams"
          :show-labels="false"
          :allow-empty="false"
          @input="selectTeam($event, false)"
        >
          <template slot="singleLabel" slot-scope="props">
            <span>
              <img :src="props.option.strTeamBadge" alt="Team Badge" width="20" height="20">
              <span>{{ props.option.strTeam }}</span>
            </span>
          </template>
          <template slot="option" slot-scope="props">
            <span>
              <img :src="props.option.strTeamBadge" alt="Team Badge" width="20" height="20">
              <span>{{ props.option.strTeam }}</span>
            </span>
          </template>
        </multiselect>

        <multiselect
          v-if="showFavorite"
          v-model="favoriteTeam"
          placeholder="Your Favorite Teams"
          label="strTeam"
          :options="favoriteTeams"
          :show-labels="false"
          :allow-empty="false"
          @input="selectTeam($event, true)"
        >
          <template slot="singleLabel" slot-scope="props">
            <span>
              <img :src="props.option.strTeamBadge" alt="Team Badge" width="20" height="20">
              <span>{{ props.option.strTeam }}</span>
            </span>
          </template>
          <template slot="option" slot-scope="props">
            <span>
              <img :src="props.option.strTeamBadge" alt="Team Badge" width="20" height="20">
              <span>{{ props.option.strTeam }}</span>
            </span>
          </template>
        </multiselect>
      </div>

      <div v-if="status === 1.5" class="info">
        <h2> Cannot get TEAM data </h2>
        <h3> Probably TheSportDB service does not provide the data</h3>
        <h3> Please select another sports/leagues</h3>
      </div>

      <div v-if="status === 2.5" class="info">
        <h2> Cannot get EVENT data </h2>
        <h3> Probably TheSportDB service does not provide the data</h3>
        <h3> Please select another sports/leagues/teams</h3>
      </div>

      <div v-if="status > 3" class="info">
        <p>Event : {{ eventinfo.strEvent }}</p>

        <div>
          <p>
            <span v-if="one_on_one">Home </span>
            <span>Team : {{ teaminfo.home.strTeam }}
              <a v-bind:href="'https://' + teaminfo.home.strWebsite" target="_blank">(Homepage)</a>
            </span>
            <span>&nbsp;|&nbsp;</span>
            <span v-if="teaminfo.home.strDescriptionEN != null">
              <button @click="teaminfo.showhomedetail=!teaminfo.showhomedetail">Detail</button>
            </span>
            <span>
              <button @click="addFavorite()">Add</button>
            </span>
          </p>
          <p v-show="teaminfo.showhomedetail">{{ homeTeamDetail }}</p>
        </div>

        <div v-if="one_on_one">
          <p>
            <span>Away Team : {{ teaminfo.away.strTeam }}
              <a v-bind:href="'https://' + teaminfo.away.strWebsite" target="_blank">(Homepage)</a>
            </span>
            <span>&nbsp;|&nbsp;</span>
            <span v-if="teaminfo.away.strDescriptionEN != null">
              <button @click="teaminfo.showawaydetail=!teaminfo.showawaydetail">Detail</button>
            </span>
          </p>
          <p v-show="teaminfo.showawaydetail">{{ awayTeamDetail }}</p>
        </div>

        <p>Location : {{ venuename }}</p>

        <div v-show="!showJST">
          <span>
            {{ timeUTC }}
          </span>
          <span>&nbsp;|&nbsp;</span>
          <span>
            <button @click="showJST=!showJST">JST</button>
          </span>
        </div>
        <div v-show="showJST">
          <span>
            {{ timeJST }}
          </span>
          <span>&nbsp;|&nbsp;</span>
          <span>
            <button @click="showJST=!showJST">UTC</button>
          </span>
        </div>

        <div class="weather">
          <p>Weather : </p>
          <img
            v-bind:src="'https://openweathermap.org/img/wn/' + weatherinfo.weather[0].icon + '@2x.png'"
            v-bind:alt="weatherinfo.weather[0].description"
            v-bind:title="weatherinfo.weather[0].main"
            width="20" height="20">
          <p>{{ weatherinfo.weather[0].main }} &ensp;|&ensp;</p>
          <button @click="showWeatherDetail=!showWeatherDetail">Detail</button>
        </div>

        <div v-show="showWeatherDetail">
          <p>Description :  {{ weatherinfo.weather[0].description }}</p>
          <p>Temperature :  {{ temp[0] }} &#8451;</p>
          <p>Feeling temperature :  {{ temp[1]}} &#8451;</p>
          <p>Atmospheric Pressure :  {{ weatherinfo.pressure }} hPa</p>
          <p>Humidity :  {{ weatherinfo.humidity }} %</p>
          <p>Clouds :  {{ weatherinfo.clouds }} %</p>
          <p>Wind speed :  {{ weatherinfo.wind_speed }}  meter/sec</p>
        </div>
      </div>

      <div id="mapContainer" class="map"></div>
    </main>
  </div>
</template>

<script>
// Set your TheSportsDB API key
const db_APIkey = process.env.VUE_APP_THESPORTSDB_KEY

// Set your Mapbox API key
const mapbox_APIKey = process.env.VUE_APP_MAPBOX_KEY

// Set your OpenWeather API key
const weather_APIKey = process.env.VUE_APP_OPENWEATHER_KEY

// Setup i18n-iso-countries
const countries = require("i18n-iso-countries");
countries.registerLocale(require("i18n-iso-countries/langs/en.json"));

import mapboxgl from 'mapbox-gl'
import Multiselect from 'vue-multiselect'

export default {
  components: {
    Multiselect
  },
  data() {
    return {
      // Mapbox config
      accessToken: mapbox_APIKey,
      mapStyle: 'mapbox://styles/mapbox/satellite-streets-v11',
      center: { lon: 138, lat: 36 },
      zoom: 3,
      map: null,
      marker: null,

      // Randering
      apiStatus: {
        'thesportdb': "processing",
        'mapbox': "processing",
        'openweather': "processing",
      },
      renderKey: {
        leagues: 1,
        teams: 2,
      },
      renderList: 2,
      status: 0,

      // Data
      allSports: [],
      sport: "",
      allLeagues: [],
      filtedLeagues: [],
      league: [],
      allTeams: [],
      selectedTeam: [],
      favoriteTeams: [],
      favoriteTeam: [],
      team: [],

      // MISC
      showFavorite: false,
      eventinfo: null,
      one_on_one: true,
      teaminfo: {
        'home': null,
        'showhomedetail': false,
        'away': null,
        'showawaydetail': false,
      },
      venuename: null,
      showJST: false,

      // Weather data
      weatherinfo: null,
      isCurrent: true,
      showWeatherDetail: false,
    };
  },
  beforeCreate() {
    fetch(`https://www.thesportsdb.com/api/v1/json/${db_APIkey}/all_leagues.php`)
    .then(response => response.json())
    .then(json => {
      // Get all leagues
      this.allLeagues = json.leagues
      const allsports = []
      for (let league in this.allLeagues) {
        allsports.push(this.allLeagues[league].strSport)
      }
      // Get all sports
      this.allSports = Array.from(new Set(allsports))
      this.status = 1
      this.apiStatus.thesportdb = 'Valld key'
    })
    .catch(error => {
      this.status = -0.5
      console.error('TheSportsDB Error:', error)
      this.apiStatus.thesportdb = 'Invalld key'
    })
  },
  mounted() {
    // Load default map
    mapboxgl.accessToken = this.accessToken

    this.map = new mapboxgl.Map({
      container: 'mapContainer',
      style: this.mapStyle,
      center: this.center,
      zoom: this.zoom
    })

    // Status change
    this.apiStatus.mapbox = 'Map loaded'
  },
  computed: {
    // Return TeamDetail
    homeTeamDetail () {
      return this.teaminfo.home.strDescriptionEN
    },
    awayTeamDetail () {
      return this.teaminfo.away.strDescriptionEN
    },

    // Calculate Time
    timeUTC () {
      return `${this.eventinfo.dateEvent} ${this.eventinfo.strTime} (UTC)`
    },
    timeJST () {
      const utc = `${this.eventinfo.dateEvent} ${this.eventinfo.strTime}`
      const unixUTC = this.unixTime(utc)
      const unixJST = unixUTC + 32400 // GMT +9

      const jstmillisecond = new Date(unixJST * 1000)
      const jstYMD = jstmillisecond.toLocaleDateString('ja', {
        year: 'numeric',
        month: '2-digit',
        day:   '2-digit'
      }).replace(/\//g, '-')
      const jstHMS = jstmillisecond.toTimeString().substr(0, 8)
      return `${jstYMD} ${jstHMS} (JST)`
    },
    // Return Temperature depending on the situation
    temp () {
      if (!this.isCurrent) {
        let temp
        let feel_temp

        const utc = `${this.eventinfo.dateEvent} ${this.eventinfo.strTime}`
        const unix = this.unixTime(utc)

        const sunrise = this.weatherinfo.sunrise
        const noon = this.unixTime(`${this.eventinfo.dateEvent} 12:00:00`)
        const sunset = this.weatherinfo.sunset
        const night = this.unixTime(`${this.eventinfo.dateEvent} 22:00:00`)

        if ( sunrise < unix && unix <= noon ) {
          // Morning
          temp = this.weatherinfo.temp.morn
          feel_temp = this.weatherinfo.feels_like.morn
        } else if ( noon < unix && unix <= sunset ) {
          // Day
          temp = this.weatherinfo.temp.day
          feel_temp = this.weatherinfo.feels_like.day
        } else if ( sunset < unix && unix <= night ) {
          // Evening
          temp = this.weatherinfo.temp.eve
          feel_temp = this.weatherinfo.feels_like.eve
        } else {
          // Night
          temp = this.weatherinfo.temp.night
          feel_temp = this.weatherinfo.feels_like.night
        }
        return [temp, feel_temp]
      } else {
        return [this.weatherinfo.temp, this.weatherinfo.feels_like]
      }
    }
  },
  methods: {
    selectSport() {
      // Reset child select
      this.league = []
      this.favoriteTeam = []
      this.apiStatus.thesportdb = 'Valld Key'
      this.apiStatus.mapbox = 'Map loaded'
      this.apiStatus.openweather = 'processing'
      this.showDefaultMap()
      this.renderKey.leagues += this.renderList

      // Filtering leagues by selected sport
      this.filtedLeagues = this.allLeagues.filter(d => d.strSport === this.sport)

      // Status update
      this.status = 2
    },

    selectLeague() {
      // Reset child select
      this.selectedTeam = []
      this.favoriteTeam = []
      this.apiStatus.thesportdb = 'Valld Key'
      this.apiStatus.mapbox = 'Map loaded'
      this.apiStatus.openweather = 'processing'
      this.showDefaultMap()
      this.renderKey.teams += this.renderList

      // Filtering teams by selected league
      const leagueID = this.league.idLeague
      fetch(`https://www.thesportsdb.com/api/v1/json/${db_APIkey}/lookup_all_teams.php?id=${leagueID}`)
      .then(response => response.json())
      .then(json => {
        this.allTeams = json.teams
        if (this.allTeams !== null) {
          // Status update
          this.status = 3
        } else {
          this.status = 1.5
        }
      })
    },

    selectTeam(event, isFavorite) {
      if (isFavorite) {
        if (this.status < 2.5) {
          this.selectedTeam = null
        } else {
          this.selectedTeam = []
        }
        this.renderKey.leagues += this.renderList
        this.renderKey.teams += this.renderList
      } else {
        this.favoriteTeam = []
      }
      this.status = 3

      // Get TeamID
      this.team = event
      const teamID = this.team.idTeam

      // Fly to location
      this.flyToLocation(teamID)
    },

    async getEventData(teamID) {
      // Reset render condition
      this.showJST = false
      this.showWeatherDetail = false

      // Check your TheSportDB plan
      let urlPram = 'eventslast.php'
      this.apiStatus.thesportdb = 'FREE Plan'

      if (db_APIkey !== '1') {
        urlPram = 'eventsnext.php'
        this.apiStatus.thesportdb = 'PAID Plan'
      }

      const response = await fetch(`https://www.thesportsdb.com/api/v1/json/${db_APIkey}/${urlPram}?id=${teamID}`)
      const json = await response.json()
      if (json.results === null) {
        this.status = 2.5
        // Reset all
        this.renderKey.leagues += this.renderList
        this.apiStatus.thesportdb = 'Data Error'
        this.apiStatus.openweather = 'processing'
        return [null, null, this.status]
      } else {
        this.eventinfo = await json.results[0]

        if (this.eventinfo.strHomeTeam === null || this.eventinfo.strAwayTeam === null) {
          this.one_on_one = false
        } else {
          this.one_on_one = true
        }
        return [this.eventinfo, this.one_on_one, this.status]
      }
    },

    async getTeamJSON(id) {
      const response = await fetch(`https://www.thesportsdb.com/api/v1/json/${db_APIkey}/lookupteam.php?id=${id}`)
      const json = await response.json()
      return json.teams[0]
    },

    async getTeamInfo(teamID) {
      const [eventInfo, one_on_one, status] = await this.getEventData(teamID)
      let target = null

      if (status !== 2.5) {
        this.teaminfo.home = this.team

        if (one_on_one) {
          // if One-on-One style, set venue name by home staduim
          const name = this.teaminfo.home.strStadium
          const place = this.teaminfo.home.strStadiumLocation
          const country = this.teaminfo.home.strCountry
          this.venuename = name + ", " + place + ", " + country
          target = name + " " + country

          // get AwayTeam info
          const awayTeamID = eventInfo.idAwayTeam
          const awayTeam = await this.getTeamJSON(awayTeamID)
          this.teaminfo.away = awayTeam
        } else {
          if (eventInfo.strVenue !== null) {
            this.venuename = eventInfo.strVenue;
            target = eventInfo.strVenue
          } else {
            this.venuename = 'Place is not found';
            target = 'Japan National Stadium';
          }
        }
        return [target, one_on_one, status]
      } else {
        return [target, one_on_one, status]
      }
    },

    async getLngLat(teamID) {
      const [target, one_on_one, status] = await this.getTeamInfo(teamID)
      const geoURLbase = `https://api.mapbox.com/geocoding/v5/mapbox.places/${encodeURIComponent(target)}.json?types=poi`
      let geoURL

      if (status !== 2.5) {
        let country_code
        if (one_on_one && this.team.strCountry !== null) {
          country_code = countries.getAlpha2Code(this.team.strCountry, "en")

          // Manually set country_code by country name
          if (typeof(country_code) === "undefined") {
            if (this.team.strCountry === "England") {
              country_code = "GB"
            } else if (this.team.strCountry === "United States") {
              country_code = "US"
            }
          }

          if (typeof(country_code) === "undefined") {
            geoURL = `${geoURLbase}&country=${country_code}&limit=1&language=en&access_token=${mapbox_APIKey}`
          } else {
            geoURL = `${geoURLbase}&limit=1&language=en&access_token=${mapbox_APIKey}`
          }
        } else {
          geoURL = `${geoURLbase}&limit=1&language=en&access_token=${mapbox_APIKey}`
        }
        const response = await fetch(geoURL)
        const json = await response.json()
        const center = json.features[0].center
        return [center, status]
      } else {
        return [null, status]
      }
    },

    async flyToLocation(teamID) {
      const [lnglat, status] = await this.getWeatherData(teamID)

      // Fly to and Add marker
      if (this.marker !== null) {
        this.marker.remove()
      }
      if (status !== 2.5) {
        this.status = 4
        // Set zoom level by Sport
        const nonStadiumSports = ['Motorsport', 'Golf', 'Cycling', 'Motorsports']
        if (!nonStadiumSports.includes(this.sport)) {
          this.zoom = 16
        } else {
          this.zoom = 13
        }

        // Fly to
        this.map.flyTo({
          center: lnglat,
          zoom: this.zoom,
          essential: true
        })

        // Status change
        this.apiStatus.mapbox = 'Show location'

        // Add marker
        this.marker = new mapboxgl.Marker()
        .setLngLat(lnglat)
        .addTo(this.map)
      } else {
        this.showDefaultMap()
      }
    },

    // Show default map
    showDefaultMap () {
      if (this.marker !== null) {
        this.marker.remove()
      }
      this.zoom = 3
      this.map.flyTo({
        center: this.center,
        zoom: this.zoom,
        speed: 3,
        curve: 2,
        essential: true
      })
    },

    // UTC to Unix time (second)
    unixTime(utc) {
      return Math.floor(Date.parse(utc) / 1000)
    },

    async getWeatherJSON(url) {
      const response = await fetch(url)
      return response.json()
    },

    // Get weather data
    async getWeatherData(teamID) {
      const [lnglat, status] = await this.getLngLat(teamID)

      if (status !== 2.5) {
        const evemtday_UnixTime = this.unixTime(this.eventinfo.dateEvent)

        let date = new Date() ;
        let today_UnixTime = Math.floor( date.getTime() / 1000 ) ;

        const json = await this.getWeatherJSON(`https://api.openweathermap.org/data/2.5/onecall?lat=${lnglat[1]}&lon=${lnglat[0]}&exclude=minutely,hourly,alerts&units=Metric&appid=${weather_APIKey}`)

        const lag = evemtday_UnixTime - today_UnixTime

        if (lag >= 0) {
          // Get weather forcast data
          this.isCurrent = false
          const lagDay = Math.min( (Math.floor(lag / (24*60*60))), 7 )
          this.weatherinfo = json.daily[lagDay]
          this.apiStatus.openweather = 'Show weather forecast'
        } else {
          // Get current weather data
          this.isCurrent = true
          this.weatherinfo = json.current
          this.apiStatus.openweather = 'Show current weather'
        }
      }
      return [lnglat, status]
    },

    // Add favorite
    addFavorite() {
      // Check if added team already exists.
      if (this.favoriteTeams.length !== 0) {
        let favo_team_list = []
        for (let i = 0; i < this.favoriteTeams.length; i++) {
          favo_team_list.push(this.favoriteTeams[i].idTeam)
        }
        if (!favo_team_list.includes(this.team.idTeam)) {
          this.favoriteTeams.push(this.team)
        }
      } else {
        this.favoriteTeams.push(this.team)
        this.showFavorite = true
      }
    }
  }
};
</script>

/* Local style */
<style scoped>
.wrapper {
  height: auto;
  width: auto;

  display: flex;
  flex-direction: column;
}

/* Header content */
header {
  height: 10vh;
  display: flex;
  flex-direction: row;
  align-items:center;
  justify-content:center;
  padding-left: 2vw;
  color: white;
  background-color: #162b32;
}
header > p {
  font-size: 4vmin;
}
header > .status {
  font-size: 1.5vmin;
  margin-left: auto;
  padding-right: 1vw;
}
header > .status > ul {
  text-align: center;
  list-style: none;
  padding-left: 0;
}

/* Main content */
main {
  flex: 1;
}
.info {
  text-align: center;
}
.weather {
  display:flex;
  align-items: center;
  justify-content:center;
}
main > .map {
  margin-bottom: auto;
  height: 70vh;
  overflow: hidden;
}
</style>
