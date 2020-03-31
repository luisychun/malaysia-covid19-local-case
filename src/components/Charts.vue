<template>
  <div>
    <p>Charts</p>
    <line-chart :data="lists" />
  </div>
</template>

<script>
export default {
  name: "Charts",
  data() {
    return {
      lists: [
        { name: "Workout", data: { "01-01-2017": 3, "2017-01-02": 4 } },
        { name: "Call parents", data: { "2017-01-01": 5, "2017-01-02": 3 } }
      ]
    };
  }
};
</script>

<style>
</style>