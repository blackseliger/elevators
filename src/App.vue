<template>

  <div class="lifts">

  <LiftPage :levels="levels"/>

  </div>
</template>

<script>
import LiftPage from "@/components/LiftPage.vue";
// import UiButton from '@/components/UiButton.vue';

export default {
  name: "App",
  components: {
    LiftPage,
  },

  data() {
    return { levels: [6, 5, 4, 3, 2, 1] };
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
