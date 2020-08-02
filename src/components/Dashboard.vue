<template>
  <v-container fluid>
    <v-row class="text-center justify-center">
      <v-col cols="12" lg="3" v-for="(category, index) in filterArray" :key="index">
        <v-card class="pa-2" outlined tile>
          <v-card-text>
            <p class="display-1 text--primary">{{ category.title }}</p>
            <p class="display-2">{{ getLatestCaseAvailable(category.title) }}</p>
            <span class="mt-2" v-if="compareCase(category.title) > 0">
              <v-icon :color="iconColor(category.title)">mdi-arrow-top-right</v-icon>
              {{ compareCase(category.title) }} as previous day
            </span>
            <span class="mt-2" v-else-if="compareCase(category.title) == 0">
              <v-icon :color="iconColor(category.title)">mdi-arrow-right</v-icon>
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
        <a
          href="https://github.com/CSSEGISandData/COVID-19"
          target="_blank"
        >JHU CSSE</a>
      </p>
    </v-row>
    <v-row class="text-center justify-center mt-4">
      <Charts :confirm="confirmProps" :death="deathProps" :recover="recoverProps" />
    </v-row>
    <v-row class="text-center justify-center mt-4">
      <State :state="stateData" />
    </v-row>
  </v-container>
</template>

<script>
import Charts from "@/components/Charts.vue";
import State from "@/components/State.vue";
export default {
  name: "Dashboard",
  components: {
    Charts,
    State
  },
  data: () => ({
    categories: [
      { title: "Confirmed" },
      { title: "Deaths" },
      { title: "Recovered" },
      { title: "States" }
    ],
    requestURL: [
      "https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_global.csv",
      "https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_deaths_global.csv",
      "https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_recovered_global.csv",
      "https://raw.githubusercontent.com/ynshung/covid-19-malaysia/master/covid-19-my-states-cases.csv"
    ],
    globalDataKey: new Object(),
    globalDataSet: new Array(),
    confirmSet: new Object(),
    deathSet: new Object(),
    recoverSet: new Object(),
    confirmProps: new Array(),
    deathProps: new Array(),
    recoverProps: new Array(),
    currentDate: "",
    previousDate: "",
    latestDateGet: "",
    latestConfirmedIndex: "",
    latestDeadIndex: "",
    latestRecoveredIndex: "",

    // State Data Set
    overallStateData: new Array(),
    latestStateData: new Object(),
    stateData: new Array()
  }),
  methods: {
    fetchCases() {
      for (let i = 0; i < this.categories.length; i++) {
        let category = this.categories[i].title,
          url = this.requestURL[i],
          papa = this.$papa,
          dataSet = new Array(),
          self = this;

        let promise = new Promise(function(resolve, reject) {
          papa.parse(url, {
            download: true,
            complete: function(results) {
              dataSet = results.data;
              if (dataSet) {
                resolve();
              } else {
                reject();
              }
            }
          });
        });

        promise
          .then(function() {
            category == "States"
              ? self.processStateData(dataSet)
              : self.processData(dataSet, category);
          })
          .catch(function(err) {
            // console.log(err);
            return err;
          });
      }
    },

    processData(dataSet, category) {
      this.globalDataKey = dataSet[0];
      let self = this;
      for (let i = 1; i < dataSet.length; i++) {
        let dataContainer = new Object();
        for (let k = 0; k < this.globalDataKey.length; k++) {
          dataContainer[this.globalDataKey[k]] = dataSet[i][k];
        }
        this.globalDataSet.push(dataContainer);
      }
      self.getMyData(this.globalDataSet, category);
    },

    getMyData(myData, category) {
      myData.forEach((my, index) => {
        let key = myData[index]["Country/Region"];
        if (key === "Malaysia") {
          let last = new String();
          let latestSet = myData[index];
          last = Object.keys(latestSet).pop();
          this.latestDateGet = last;
          if (category === "Confirmed") {
            this.confirmSet = myData[index];
            this.latestConfirmedIndex = this.confirmSet[last];
            this.confirmProps = Object.entries(this.confirmSet).splice(4);
          } else if (category === "Deaths") {
            this.deathSet = myData[index];
            this.latestDeadIndex = this.deathSet[last];
            this.deathProps = Object.entries(this.deathSet).splice(4);
          } else {
            this.recoverSet = myData[index];
            this.latestRecoveredIndex = this.recoverSet[last];
            this.recoverProps = Object.entries(this.recoverSet).splice(4);
          }
        }
      });
    },

    getCurrentDate() {
      let today = new Date();
      today.setDate(today.getDate() - 1);
      let dd = today.getDate();
      let mm = today.getMonth() + 1;
      let yy = today
        .getFullYear()
        .toString()
        .substr(-2);
      this.currentDate = `${mm}/${dd}/${yy}`;
    },

    getPreviousDate() {
      let today = new Date();
      today.setDate(today.getDate() - 2);
      let dd = today.getDate();
      let mm = today.getMonth() + 1;
      let yy = today
        .getFullYear()
        .toString()
        .substr(-2);
      this.previousDate = `${mm}/${dd}/${yy}`;
    },

    getLatestCaseAvailable(title) {
      if (title === "Confirmed") {
        return this.latestConfirmedIndex;
      } else if (title === "Deaths") {
        return this.latestDeadIndex;
      } else {
        return this.latestRecoveredIndex;
      }
    },

    compareCase(title) {
      let currentCase = "";
      let previousCase = "";
      if (title == "Confirmed") {
        currentCase = this.confirmSet[this.currentDate];
        previousCase = this.confirmSet[this.previousDate];
      } else if (title === "Deaths") {
        currentCase = this.deathSet[this.currentDate];
        previousCase = this.deathSet[this.previousDate];
      } else {
        currentCase = this.recoverSet[this.currentDate];
        previousCase = this.recoverSet[this.previousDate];
      }

      if (currentCase == null || previousCase == null) {
        return "Data not available";
      }

      if (parseInt(currentCase) === parseInt(previousCase)) {
        return 0;
      } else {
        let diff = "";
        diff = currentCase - previousCase;
        return parseInt(diff);
      }
    },

    iconColor(title) {
      let color = parseInt(this.compareCase(title)) > 0 ? "red" : "green";
      return color;
    },

    // State Data
    processStateData(dataSet) {
      let key = dataSet[0];
      for (let i = 1; i < dataSet.length - 1; i++) {
        let dataContainer = new Object();
        for (let k = 1; k < key.length; k++) {
          dataContainer[key[k]] = dataSet[i][k];
        }
        this.overallStateData.push(dataContainer);
      }
      this.latestStateData = this.overallStateData[
        this.overallStateData.length - 1
      ];
      this.stateData = Object.entries(this.latestStateData);
    }
  },

  computed: {
    filterArray() {
      let lists = [
        { title: "Confirmed" },
        { title: "Deaths" },
        { title: "Recovered" }
      ];
      return lists;
    }
  },

  created() {
    this.fetchCases();
    this.getCurrentDate();
    this.getPreviousDate();
  }
};
</script>
