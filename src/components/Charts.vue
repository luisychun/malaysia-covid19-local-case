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
    <v-row>
      <line-chart :data="fetchData(caseFilter)" :library="options"></line-chart>
    </v-row>
  </div>
</template>

<script>
export default {
  name: 'Charts',
  props: ['confirm', 'death', 'recover'],
  data() {
    return {
      caseFilter: 'All',
      items: ['All', 'Confirmed', 'Deaths', 'Recovered'],
      options: {
        scales: {
          yAxes: [
            {
              ticks: {
                display: true,
              },
            },
          ],
          xAxes: [
            {
              type: 'time',
              // ticks: {
              //   min: '12/1/20',
              // },
            },
          ],
        },
      },
    }
  },
  methods: {
    changeFilter(ans) {
      this.caseFilter = ans
      this.fetchData(this.caseFilter)
    },
    fetchData(filter) {
      let list = new Array()
      let confirmList = [],
        deathList = [],
        recoverList = []
      list = [
        { name: 'Confirm', data: this.confirm },
        { name: 'Deaths', data: this.death },
        { name: 'Recover', data: this.recover },
      ]
      confirmList = [{ name: 'Confirm', data: this.confirm }]
      deathList = [{ name: 'Deaths', data: this.death }]
      recoverList = [{ name: 'Recover', data: this.recover }]
      if (filter == 'All') {
        return list
      } else if (filter == 'Confirmed') {
        return confirmList
      } else if (filter == 'Deaths') {
        return deathList
      } else {
        return recoverList
      }
    },
  },
  created() {
    this.fetchData(this.caseFilter)
    if (
      /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(
        navigator.userAgent
      )
    ) {
      this.options.scales.yAxes[0].ticks.display = false
      this.options.scales.xAxes[0].ticks.min = '6/1/20'
    }
  },
}
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
}</style
>>
