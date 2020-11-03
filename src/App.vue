<template>
  <div id="app">
    <parse />
  </div>
</template>

<script>
import Parse from "./components/Parse.vue";

export default {
  name: "app",
  components: {
    Parse
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
