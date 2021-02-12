<template>
  <v-container fluid>
    <v-row class="text-center justify-center">
      <v-col
        cols="12"
        lg="3"
        v-for="(category, index) in filterArray"
        :key="index"
      >
        <v-card class="pa-2" outlined tile>
          <v-card-text>
            <p class="display-1 text--primary">{{ category.title }}</p>
            <p class="display-2">
              {{ getLatestCaseAvailable(category.title) }}
            </p>
            <span class="mt-2" v-if="compareCase(category.title) > 0">
              <v-icon :color="iconColor(category.title)"
                >mdi-arrow-top-right</v-icon
              >
              {{ compareCase(category.title) }} as previous day
            </span>
            <span class="mt-2" v-else-if="compareCase(category.title) == 0">
              <v-icon :color="iconColor(category.title)"
                >mdi-arrow-right</v-icon
              >
              {{ compareCase(category.title) }} as previous day
            </span>
            <span class="mt-2" v-else>
              <v-icon>mdi-window-close</v-icon>
              {{ compareCase(category.title) }}
            </span>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
    <v-row class="text-center justify-center mt-4">
      <p>
        Last updated: {{ latestDateGet }} from
        <a href="https://github.com/CSSEGISandData/COVID-19" target="_blank"
          >JHU CSSE</a
        >
      </p>
    </v-row>
    <v-row class="text-center justify-center mt-4 d-none d-lg-flex">
      <Charts
        :confirm="confirmProps"
        :death="deathProps"
        :recover="recoverProps"
      />
    </v-row>
    <v-row class="text-center justify-center mt-4">
      <State :state="stateData" />
    </v-row>
    <v-row class="text-center justify-center mt-4">
      <p class="text-center justify-content-center mt-4">
        Last updated: {{ latestDateGetState }} from
        <a href="https://github.com/ynshung/covid-19-malaysia" target="_blank"
          >ynshung GitHub repos</a
        >
      </p>
    </v-row>
  </v-container>
</template>

<script>
import Charts from '@/components/Charts.vue'
import State from '@/components/State.vue'
export default {
  name: 'Dashboard',
  components: {
    Charts,
    State,
  },
  data: () => ({
    categories: [
      { title: 'Confirmed' },
      { title: 'Deaths' },
      { title: 'Recovered' },
      { title: 'States' },
    ],
    requestURL: [
      'https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_global.csv',
      'https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_deaths_global.csv',
      'https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_recovered_global.csv',
      'https://raw.githubusercontent.com/ynshung/covid-19-malaysia/master/covid-19-my-states-cases.csv',
    ],
    dataKey: new Object(), // Province/State, Country/Region, Lat, Long, list of date
    dataSet: new Array(),
    confirmSet: new Object(),
    deathSet: new Object(),
    recoverSet: new Object(),
    // Props pass to chart component
    confirmProps: new Array(),
    deathProps: new Array(),
    recoverProps: new Array(),
    // Current and previous date
    currentDate: '',
    previousDate: '',
    latestDateGet: '',
    latestDateGetState: '',
    latestConfirmedIndex: '',
    latestDeadIndex: '',
    latestRecoveredIndex: '',

    // State Data Set
    overallStateData: new Array(),
    latestStateData: new Object(),
    stateData: new Array(), // Props pass to state component
  }),
  methods: {
    fetchCases() {
      for (let i = 0; i < this.categories.length; i++) {
        let category = this.categories[i].title,
          url = this.requestURL[i],
          papa = this.$papa,
          dataSet = new Array(), // Store all the data
          self = this

        let promise = new Promise((resolve, reject) => {
          papa.parse(url, {
            download: true,
            complete: (results) => {
              dataSet = results.data
              if (dataSet) {
                resolve()
              } else {
                reject()
              }
            },
          })
        })

        promise
          .then(() => {
            category == 'States'
              ? self.processStateData(dataSet)
              : self.processData(dataSet, category) // Process Malaysia's State Data
          })
          .catch((err) => {
            return err
          })
      }
    },

    processData(dataSet, category) {
      this.dataKey = dataSet[0]
      let self = this
      for (let i = 1; i < dataSet.length; i++) {
        let dateMapCases = new Object()
        for (let k = 0; k < this.dataKey.length; k++) {
          dateMapCases[this.dataKey[k]] = dataSet[i][k]
        }
        this.dataSet.push(dateMapCases)
      }
      self.getMyData(this.dataSet, category)
    },

    getMyData(myData, category) {
      myData.forEach((my, index) => {
        let key = myData[index]['Country/Region']
        if (key === 'Malaysia') {
          let last = new String()
          let latestSet = myData[index]
          last = Object.keys(latestSet).pop()
          this.latestDateGet = last
          if (category === 'Confirmed') {
            this.confirmSet = myData[index]
            this.latestConfirmedIndex = this.confirmSet[last]
            this.confirmProps = Object.entries(this.confirmSet).splice(4) // Pass only date with case data to chart props
          } else if (category === 'Deaths') {
            this.deathSet = myData[index]
            this.latestDeadIndex = this.deathSet[last]
            this.deathProps = Object.entries(this.deathSet).splice(4)
          } else {
            this.recoverSet = myData[index]
            this.latestRecoveredIndex = this.recoverSet[last]
            this.recoverProps = Object.entries(this.recoverSet).splice(4)
          }
        }
      })
    },

    getDate(date) {
      let today = new Date()
      today.setDate(today.getDate() - date)
      let dd = today.getDate()
      let mm = today.getMonth() + 1
      let yy = today
        .getFullYear()
        .toString()
        .substr(-2)
      date == 1
        ? (this.currentDate = `${mm}/${dd}/${yy}`)
        : (this.previousDate = `${mm}/${dd}/${yy}`)
    },

    getLatestCaseAvailable(title) {
      return title === 'Confirmed'
        ? this.latestConfirmedIndex
        : title === 'Deaths'
        ? this.latestDeadIndex
        : this.latestRecoveredIndex
    },

    compareCase(title) {
      let currentCase = ''
      let previousCase = ''
      if (title == 'Confirmed') {
        currentCase = this.confirmSet[this.currentDate]
        previousCase = this.confirmSet[this.previousDate]
      } else if (title === 'Deaths') {
        currentCase = this.deathSet[this.currentDate]
        previousCase = this.deathSet[this.previousDate]
      } else {
        currentCase = this.recoverSet[this.currentDate]
        previousCase = this.recoverSet[this.previousDate]
      }

      if (currentCase == null || previousCase == null) {
        return 'Latest case not available'
      }

      if (parseInt(currentCase) === parseInt(previousCase)) {
        return 0
      } else {
        let diff = ''
        diff = currentCase - previousCase
        return parseInt(diff)
      }
    },

    iconColor(title) {
      let color = ''
      if (title !== 'Recovered') {
        color = parseInt(this.compareCase(title)) > 0 ? 'red' : 'green'
      } else {
        color = parseInt(this.compareCase(title)) > 0 ? 'green' : 'red'
      }

      return color
    },

    // State Data
    processStateData(dataSet) {
      let key = dataSet[0]
      this.latestDateGetState = dataSet[dataSet.length - 2][0]
      for (let i = 1; i < dataSet.length - 1; i++) {
        let dateMapCases = new Object()
        for (let k = 1; k < key.length; k++) {
          dateMapCases[key[k]] = dataSet[i][k]
        }
        this.overallStateData.push(dateMapCases)
      }
      this.latestStateData = this.overallStateData[
        this.overallStateData.length - 1
      ]
      this.stateData = Object.entries(this.latestStateData)
    },
  },

  computed: {
    filterArray() {
      let lists = [
        { title: 'Confirmed' },
        { title: 'Deaths' },
        { title: 'Recovered' },
      ]
      return lists
    },
  },

  created() {
    this.fetchCases()
    this.getDate(1) // Get currect date
    this.getDate(2) // Get previous date
  },
}
</script>
