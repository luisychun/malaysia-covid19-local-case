<template>
  <div>
    <v-row class="justify-center text-center mt-4">
      <p class="display-1 text--primary">State Data</p>
    </v-row>
    <bar-chart :data="state"></bar-chart>
  </div>
</template>

<script>
export default {
  name: "State",
  props: ["state"]
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