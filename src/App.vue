<template>
  <OpenStreetMap />
</template>

<script>
import OpenStreetMap from "@/components/OpenStreetMap/OpenStreetMap.vue";

export default {
  name: "App",
  components: {
    OpenStreetMap,
  },
};
</script>

<style lang="scss">
body {
  margin: 0;
  padding: 0;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 0;
  padding: 0;
}
</style>
