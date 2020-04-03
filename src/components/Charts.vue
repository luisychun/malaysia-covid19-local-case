<template>
  <div>
    <area-chart :data="fetchData()" width="1000px" height="600px"></area-chart>
  </div>
</template>

<script>
export default {
  name: "Charts",
  props: ["confirm", "death", "recover"],
  data() {
    return {};
  },
  methods: {
    fetchData() {
      let list = new Array();
      list = [
        { name: "Confirm", data: this.confirm },
        { name: "Death", data: this.death },
        { name: "Recover", data: this.recover }
      ];
      return list;
    }
  },
  created() {
    this.fetchData();
  }
};
</script>

<style scoped>
#chart-1 {
  position: relative;
  width: 60vw !important;
  height: 60vh !important;
}
</style>>
