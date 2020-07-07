<template>
    <v-container fluid>
    <v-row class="text-center justify-center">
      <v-col cols="12" lg="3" v-for="(item, key, index) in latestData" :key="index">
        <v-card class="pa-2" outlined tile>
          <v-card-text>
            <p class="display-1 text--primary">{{ key }}</p>
            <p class="text--primary">{{ item }}</p>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
    <v-row class="text-center justify-center">
        <bar-chart :data="graphData" :library="options" width="80vw" height="70vh"></bar-chart>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: "State",
  data() {
    return {
      url: 'https://raw.githubusercontent.com/ynshung/covid-19-malaysia/master/covid-19-my-states-cases.csv',
      overallDataSet: new Array(),
      latestData: new Object(),
      graphData: new Array()
    }
  },

  methods: {
    getStateData() {
      let url = this.url
      let papa = this.$papa
      let processData = this.processData
      let dataSet = new Array()

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
          console.log(dataSet)
          processData(dataSet)
        })
        .catch(function(err) {
          // console.log(err);
          return err;
        });
    },

    processData(dataSet) {
      let key = dataSet[0];
      console.log(key)
      for(let i= 1; i<dataSet.length - 1; i++) {
        let dataContainer = new Object();
          for (let k = 1; k < key.length; k++) {
            dataContainer[key[k]] = dataSet[i][k];
          }
          this.overallDataSet.push(dataContainer);
      }
      this.latestData = this.overallDataSet[this.overallDataSet.length - 1]
      console.log(this.latestData)

      this.graphData = Object.entries(this.latestData)
      console.log(this.graphData)
      
    }
  },

  created() {
    this.getStateData()
  }
}
</script>

<style>

</style>