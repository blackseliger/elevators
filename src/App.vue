<template>
  <LiftPage :levels="levels"/>
</template>

<script>
import LiftPage from "./components/LiftPage.vue";

export default {
  name: "App",
  components: {
    LiftPage,
  },

  data() {
    return { levels: [5, 4, 3, 2, 1] };
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
