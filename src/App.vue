<template>
  <v-app>
    <v-app-bar app color="#263238" dark>
      <v-btn small text>
        <span class="mr-2">Malaysia Covid-19 Cases</span>
      </v-btn>
      <v-spacer></v-spacer>
      <v-btn
        small
        href="https://github.com/luisychun/mys-covid-local-case/"
        target="_blank"
        text
      >
        <span class="mr-2">GitHub</span>
        <v-icon>mdi-open-in-new</v-icon>
      </v-btn>
    </v-app-bar>

    <v-content>
      <router-view></router-view>
    </v-content>
  </v-app>
</template>

<script>
export default {
  name: "App",

  components: {},

  data: () => ({
    //
  })
};
</script>
