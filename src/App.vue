<template>
  <PlayerMatches />
</template>

<script>
import PlayerMatches from "./components/PlayerMatches.vue";

export default {
  name: "App",
  components: {
    PlayerMatches,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  margin-top: 60px;
}
</style>
