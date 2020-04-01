<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12" lg="4" v-for="category in categories" :key="category.title">
        <v-card class="pa-2" outlined tile>
          <v-card-text>
            <p class="display-1 text--primary">{{ category.title }} Case</p>
            <div class="display-2 text--primary">{{ showLatestCase(category.title) }}</div>
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
      { title: "Confirmed", color: "warning" },
      { title: "Dead", color: "danger" },
      { title: "Recover", color: "primary" }
    ],
    globalDataKey: new Object(),
    globalDataSet: new Array(),
    confirmSet: new Object(),
    deathSet: new Object(),
    recoverSet: new Object(),
    fetechURL: [
      "https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_global.csv",
      "https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_deaths_global.csv",
      "https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_recovered_global.csv"
    ],
    currectDate: "",
    confirmProps: new Array(),
    deathProps: new Array(),
    recoverProps: new Array()
  }),
  methods: {
    fetchConfirmCase() {
      let category = "Confirm";
      let papa = this.$papa,
        url = this.fetechURL[0],
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
          console.log(err);
        });
    },

    fetchDeathCase() {
      let category = "Death";
      let papa = this.$papa,
        url = this.fetechURL[1],
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
          console.log(err);
        });
    },

    fetchRecoverCase() {
      let category = "Recover";
      let papa = this.$papa,
        url = this.fetechURL[2],
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
          console.log(err);
        });
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

      // console.log(this.confirmSet);
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

    showLatestCase(title) {
      if (title === "Confirmed") {
        return this.confirmSet[this.currectDate];
      } else if (title === "Dead") {
        return this.deathSet[this.currectDate];
      } else {
        return this.recoverSet[this.currectDate];
      }
    }
  },

  created() {
    this.fetchConfirmCase();
    this.fetchDeathCase();
    this.fetchRecoverCase();
    this.getCurrectDate();
  }
};
</script>
