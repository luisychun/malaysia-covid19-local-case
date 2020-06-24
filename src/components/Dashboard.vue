<template>
  <v-container fluid>
    <v-row class="text-center justify-center">
      <v-col cols="12" lg="3" v-for="(category, index) in categories" :key="index">
        <v-card class="pa-2" outlined tile>
          <v-card-text>
            <p class="display-1 text--primary">{{ category.title }}</p>
            <p class="display-2">{{ getLatestCaseAvailable(category.title) }}</p>
            <span class="mt-2">
              <v-icon
                v-if="compareCase(category.title) > 0"
                :color="iconColor(category.title)"
              >mdi-arrow-top-right</v-icon>
              <v-icon
                v-else-if="compareCase(category.title) == 0"
                :color="iconColor(category.title)"
              >mdi-arrow-right</v-icon>
              <v-icon v-else>mdi-window-close</v-icon>
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
  </v-container>
</template>

<script>
import Charts from "@/components/Charts.vue";
export default {
  name: "Dashboard",
  components: {
    Charts
  },
  data: () => ({
    categories: [
      { title: "Confirmed", color: "warning" },
      { title: "Death", color: "danger" },
      { title: "Recovered", color: "primary" }
    ],
    fetechURL: [
      "https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_global.csv",
      "https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_deaths_global.csv",
      "https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_recovered_global.csv"
    ],
    globalDataKey: new Object(),
    globalDataSet: new Array(),
    confirmSet: new Object(),
    deathSet: new Object(),
    recoverSet: new Object(),
    confirmProps: new Array(),
    deathProps: new Array(),
    recoverProps: new Array(),
    currectDate: "",
    previousDate: "",
    latestDateGet: "",
    latestConfirmedIndex: "",
    latestDeadIndex: "",
    latestRecoveredIndex: ""
  }),
  methods: {
    fetchCase() {
      for (let i = 0; i < this.categories.length; i++) {
        let category = this.categories[i].title,
          url = this.fetechURL[i],
          papa = this.$papa,
          processData = this.processData,
          dataSet = new Array();

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
            processData(dataSet, category);
          })
          .catch(function(err) {
            // console.log(err);
            return err;
          });
      }
    },

    processData(dataSet, category) {
      let getMyData = this.getMyData;
      this.globalDataKey = dataSet[0];
      for (let i = 1; i < dataSet.length; i++) {
        let dataContainer = new Object();
        for (let k = 0; k < this.globalDataKey.length; k++) {
          dataContainer[this.globalDataKey[k]] = dataSet[i][k];
        }
        this.globalDataSet.push(dataContainer);
      }
      getMyData(this.globalDataSet, category);
    },

    getMyData(myData, category) {
      myData.forEach((my, index) => {
        let key = myData[index]["Country/Region"];
        if (key === "Malaysia") {
          let last = "";
          if (category === "Confirmed") {
            this.confirmSet = myData[index];
            last = Object.keys(this.confirmSet).pop();
            this.latestConfirmedIndex = this.confirmSet[last];
            this.confirmProps = Object.entries(this.confirmSet);
            this.confirmProps = this.confirmProps.splice(4);
          } else if (category === "Death") {
            this.deathSet = myData[index];
            last = Object.keys(this.deathSet).pop();
            this.latestDeadIndex = this.deathSet[last];
            this.deathProps = Object.entries(this.deathSet);
            this.deathProps = this.deathProps.splice(4);
          } else {
            this.recoverSet = myData[index];
            last = Object.keys(this.recoverSet).pop();
            this.latestRecoveredIndex = this.recoverSet[last];
            this.recoverProps = Object.entries(this.recoverSet);
            this.recoverProps = this.recoverProps.splice(4);
          }
          this.latestDateGet = last;
        }
      });
    },

    getCurrectDate() {
      let today = new Date();
      today.setDate(today.getDate() - 1);
      let dd = today.getDate();
      let mm = today.getMonth() + 1;
      let yy = today
        .getFullYear()
        .toString()
        .substr(-2);
      this.currectDate = `${mm}/${dd}/${yy}`;
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

    showLatestCase(title) {
      if (title === "Confirmed") {
        return this.confirmSet[this.currectDate];
      } else if (title === "Death") {
        return this.deathSet[this.currectDate];
      } else {
        return this.recoverSet[this.currectDate];
      }
    },

    getLatestCaseAvailable(title) {
      if (title === "Confirmed") {
        return this.latestConfirmedIndex;
      } else if (title === "Death") {
        return this.latestDeadIndex;
      } else {
        return this.latestRecoveredIndex;
      }
    },

    compareCase(title) {
      let currentCase = "";
      let previousCase = "";
      if (title == "Confirmed") {
        currentCase = this.confirmSet[this.currectDate];
        previousCase = this.confirmSet[this.previousDate];
      } else if (title === "Death") {
        currentCase = this.deathSet[this.currectDate];
        previousCase = this.deathSet[this.previousDate];
      } else {
        currentCase = this.recoverSet[this.currectDate];
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
    }
  },

  computed: {},

  created() {
    this.fetchCase();
    this.getCurrectDate();
    this.getPreviousDate();
  }
};
</script>
