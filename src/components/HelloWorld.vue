<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="4" v-for="category in categories" :key="category.title">
        <v-card class="pa-2" outlined tile>
          <v-card-text>
            <p class="display-1 text--primary">{{ category.title }} Case</p>
            <p>adjective</p>
            <div class="text--primary">
              well meaning and kindly.
              <br />"a benevolent smile"
            </div>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: "HelloWorld",

  data: () => ({
    categories: [
      { title: "Confirmed", color: "warning" },
      { title: "Dead", color: "danger" },
      { title: "Recover", color: "primary" }
    ],
    url:
      "https://raw.githubusercontent.com/CSSEGISandData/COVID-19/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_confirmed_global.csv",
    globalDataKey: new Object(),
    globalDataSet: new Array()
  }),
  methods: {
    fetchData() {
      let papa = this.$papa,
        url = this.url,
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
          processData(dataSet);
        })
        .catch(function(err) {
          console.log(err);
        });
    },

    processData(dataSet) {
      let getMyData = this.getMyData;
      this.globalDataKey = dataSet[0];
      for (let i = 1; i < dataSet.length; i++) {
        let dataContainer = new Object();
        for (let k = 0; k < this.globalDataKey.length; k++) {
          dataContainer[this.globalDataKey[k]] = dataSet[i][k];
        }
        this.globalDataSet.push(dataContainer);
      }
      getMyData(this.globalDataSet);
    },

    getMyData(myData) {
      let data = new Object();
      myData.forEach((my, index) => {
        let key = myData[index]["Country/Region"];
        if (key === "Malaysia") {
          data = myData[index];
        }
      });
      console.log(data);
    }
  },

  created() {
    this.fetchData();
  }
};
</script>
