<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <div class="d-flex align-center">
        <v-btn text>
          <span class="mr-2">Malaysia Covid-19 Local Cases</span>
        </v-btn>
      </div>

      <v-spacer></v-spacer>

      <v-btn href="https://github.com/luisychun/malaysia-covid19-local-case" target="_blank" text>
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
