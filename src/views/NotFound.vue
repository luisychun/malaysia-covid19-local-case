<template>
  <div class="notFound">
    <div class="container fluid">
      <h1>Your requested page is not found</h1>
      <v-btn small text>
        <router-link :to="{ name: 'Home' }">
          <span class="mr-2">Home</span>
        </router-link>
      </v-btn>
    </div>
  </div>
</template>

<script>
export default {
  name: "NotFound"
};
</script>

<style>
</style>