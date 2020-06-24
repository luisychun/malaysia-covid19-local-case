<template>
  <div>
    <v-row class="justify-center text-center">
      <v-col cols="12" lg="3">
        <v-select
          :items="items"
          item-value="items"
          label="Filter Case"
          outlined
          @change="changeFilter"
        ></v-select>
      </v-col>
    </v-row>
    <line-chart :data="fetchData(caseFilter)" :min="1" xmin="6/1/20"></line-chart>
  </div>
</template>

<script>
export default {
  name: "Charts",
  props: ["confirm", "death", "recover"],
  data() {
    return {
      caseFilter: "All",
      items: ["All", "Confirmed", "Death", "Recovered"]
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
</style>>
