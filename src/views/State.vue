<template>
  <div class="state">
    <State />
  </div>
</template>

<script>
// @ is an alias to /src
import State from "@/components/State.vue";

export default {
  name: "Home",
  components: {
    State
  }
};
</script>
