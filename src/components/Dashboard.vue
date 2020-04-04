<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12" lg="4" v-for="category in categories" :key="category.title">
        <v-card class="pa-2" outlined tile>
          <v-card-text>
            <p class="display-1 text--primary">{{ category.title }} Case</p>
            <div class="display-2 text--primary">{{ showLatestCase(category.title) }}</div>
            <span class="mt-2">
              <v-icon v-if="compareCase(category.title) > 0" :color="iconColor">mdi-arrow-top-right</v-icon>
              <v-icon v-else :color="iconColor">mdi-arrow-right</v-icon>
              {{ compareCase(category.title) }}
            </span>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
    <v-row class="text-center justify-center">
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
      { title: "Confirm", color: "warning" },
      { title: "Death", color: "danger" },
      { title: "Recover", color: "primary" }
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
    comparePrevious: ""
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
          if (category === "Confirm") {
            this.confirmSet = myData[index];
            this.confirmProps = Object.entries(this.confirmSet);
            this.confirmProps = this.confirmProps.splice(4);
          } else if (category === "Death") {
            this.deathSet = myData[index];
            this.deathProps = Object.entries(this.deathSet);
            this.deathProps = this.deathProps.splice(4);
          } else {
            this.recoverSet = myData[index];
            this.recoverProps = Object.entries(this.recoverSet);
            this.recoverProps = this.recoverProps.splice(4);
          }
        }
      });
    },

    getCurrectDate() {
      let today = new Date();
      let dd = today.getDate() - 1;
      let mm = today.getMonth() + 1;
      let yy = today
        .getFullYear()
        .toString()
        .substr(-2);
      if (dd == 0) {
        dd += 1;
      }
      this.currectDate = `${mm}/${dd}/${yy}`;
    },

    getPreviousDate() {
      let today = new Date();
      let dd = today.getDate() - 2;
      let mm = today.getMonth() + 1;
      let yy = today
        .getFullYear()
        .toString()
        .substr(-2);
      if (dd == 0) {
        dd += 1;
      }
      this.previousDate = `${mm}/${dd}/${yy}`;
    },

    showLatestCase(title) {
      if (title === "Confirm") {
        return this.confirmSet[this.currectDate];
      } else if (title === "Death") {
        return this.deathSet[this.currectDate];
      } else {
        return this.recoverSet[this.currectDate];
      }
    },

    compareCase(title) {
      let currectCase = "";
      let previousCase = "";
      if (title == "Confirm") {
        currectCase = this.confirmSet[this.currectDate];
        previousCase = this.confirmSet[this.previousDate];
      } else if (title === "Death") {
        currectCase = this.deathSet[this.currectDate];
        previousCase = this.deathSet[this.previousDate];
      } else {
        currectCase = this.recoverSet[this.currectDate];
        previousCase = this.recoverSet[this.previousDate];
      }
      if (currectCase === previousCase) {
        return 0;
      } else {
        let diff = "";
        diff = currectCase - previousCase;
        return diff;
      }
    }
  },

  computed: {
    iconColor() {
      let color = this.compareCase > "0" ? "red" : "green";
      return color;
    }
  },

  created() {
    this.fetchCase();
    this.getCurrectDate();
    this.getPreviousDate();
  }
};
</script>
