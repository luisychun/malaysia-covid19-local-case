<template>
  <div>
    <line-chart :data="fetchData()" :min="1"></line-chart>
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
#chart-1,
#chart-2 {
  position: relative;
  width: 80vw !important;
  height: 80vh !important;
}
</style>>
