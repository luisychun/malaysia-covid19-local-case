<template>
  <div>
    <v-row class="justify-center text-center">
      <v-col cols="9" lg="3">
        <v-select
          :items="items"
          item-value="items"
          label="Filter Case"
          outlined
          @change="changeFilter"
        ></v-select>
      </v-col>
    </v-row>
    <line-chart :data="fetchData(caseFilter)" :min="1" xmin="6/1/20" :library="options"></line-chart>
  </div>
</template>

<script>
export default {
  name: "Charts",
  props: ["confirm", "death", "recover"],
  data() {
    return {
      caseFilter: "All",
      items: ["All", "Confirmed", "Death", "Recovered"],
      options: {
        scales: {
          yAxes: [
            {
              ticks: {
                display: false
              }
            }
          ]
        }
      }
    };
  },
  methods: {
    changeFilter(ans) {
      this.caseFilter = ans;
      this.fetchData(this.caseFilter);
    },
    fetchData(filter) {
      let list = new Array();
      let confirmList = [],
        deathList = [],
        recoverList = [];
      list = [
        { name: "Confirm", data: this.confirm },
        { name: "Death", data: this.death },
        { name: "Recover", data: this.recover }
      ];
      confirmList = [{ name: "Confirm", data: this.confirm }];
      deathList = [{ name: "Death", data: this.death }];
      recoverList = [{ name: "Recover", data: this.recover }];
      if (filter == "All") {
        return list;
      } else if (filter == "Confirmed") {
        return confirmList;
      } else if (filter == "Death") {
        return deathList;
      } else {
        return recoverList;
      }
    }
  },
  created() {
    this.fetchData(this.caseFilter);
  }
};
</script>

<style scoped>
#chart-1,
#chart-2 {
  position: relative;
  width: 80vw !important;
  height: 80vh !important;
}

@media (max-width: 768px) {
  #chart-1,
  #chart-2 {
    width: 90vw !important;
  }
}
</style>>
